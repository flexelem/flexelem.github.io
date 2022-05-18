---
title:  Create a Cognito User Pool with AWS CDK
author: buraktas
layout: post
permalink: /create-cognito-user-pool-aws-cdk/
dsq_needs_sync:
  - 1
categories:
  - aws
  - aws-cdk
tags:
  - aws-cdk
  - aws
  - cognito-userpool
comments: true
---

AWS Cognito User Pool is a user directory which provides sign-up and sign-in functionalities for your users. In this tutorial we will use User/Password Auth Flow managed by a Cognito App Client. A Cognito App Client is an application client which provides clients to call unauthenticated endpoints like sign-up, sign-in, forgot password etc. Without it users only can be added manually from AWS Console in other words by admins.

<!--more-->

Here are the steps we will follow;
- Create Cognito User Pool
- Create Cognito App Client with USER_PASSWORD_AUTH Flow
- Sign-up a user from command line with `aws.cognito-idp`
- Login with registered user and verify that we get a successful response which includes JWT tokens.

<br/>

## CDK Code of Cognito UserPool with App Client

There are two constructs we will use which are [`UserPool`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cognito.UserPool.html) and [`UserPoolClient`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_cognito.UserPoolClient.html)

```python
from aws_cdk import (
    core,
    aws_cognito as cognito
)


class CognitoUserPoolStack(core.Stack):
    def __init__(self,
                 scope: core.Construct,
                 id: str,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        cognito_user_pool = cognito.UserPool(self, "awesome-user-pool",
            user_pool_name="awesome-user-pool",
            sign_in_aliases=cognito.SignInAliases(
                email=True
            ),
            self_sign_up_enabled=True,
            auto_verify=cognito.AutoVerifiedAttrs(
                email=True  # This is True by default if email is defined in SignInAliases
            ),
            user_verification=cognito.UserVerificationConfig(
                email_subject="You need to verify your email",
                email_body="Thanks for signing up Your verification code is {####}",  # This placeholder is a must if code is selected as preferred verification method
                email_style=cognito.VerificationEmailStyle.CODE
            ),
            standard_attributes=cognito.StandardAttributes(
                family_name=cognito.StandardAttribute(
                    required=True,
                    mutable=False,
                ),
                address=cognito.StandardAttribute(
                    required=False,
                    mutable=True
                )
            ),
            custom_attributes={
                "tenant_id": cognito.StringAttribute(min_len=10, max_len=15, mutable=False),
                "created_at": cognito.DateTimeAttribute(),
                "employee_id": cognito.NumberAttribute(min=1, max=100, mutable=False),
                "is_admin": cognito.BooleanAttribute(mutable=True),
            },
            password_policy=cognito.PasswordPolicy(
                min_length=8,
                require_lowercase=True,
                require_uppercase=True,
                require_digits=True,
                require_symbols=True
            ),
            account_recovery=cognito.AccountRecovery.EMAIL_ONLY,
            removal_policy=core.RemovalPolicy.DESTROY
        )

        app_client = cognito_user_pool.add_client(
            "awesome-app-client",
            user_pool_client_name="awesome-app-client",
            auth_flows=cognito.AuthFlow(
                user_password=True
            )
        )
```

Now lets go over with the properties of UserPool construct we created above;

- `user_pool_name`: Property for defining the name of Cognito UserPool resource. Though this is not a good practice explained [here](https://aws.amazon.com/premiumsupport/knowledge-center/cloudformation-custom-name/#:~:text=AWS%20CloudFormation%20doesn't%20replace,before%20you%20update%20a%20stack.)
- [`sign_in_aliases`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/SignInAliases.html#aws_cdk.aws_cognito.SignInAliases): A list of identifiers that a user can sign-in with. For example; email, phone, username etc.
- `self_sign_up_enabled`: Allows users to sign-up. If this is set to False then users only can be registered via AWS Console or by [`sign-up`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/sign-up.html)
- [`auto_verify`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/AutoVerifiedAttrs.html): Makes Cognito to automatically verify given attributes by sending a verification code. Only email or phone can be configured for auto verify. Additionally, if these are defined in sign_in_aliases section then Cognito will auto verify them by default. However, if one them is set to False, then we have to use admin apis for verifying user's attributes by [`admin-update-user-attributes`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/admin-update-user-attributes.html)
- [`user_verification`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/UserVerificationConfig.html): Configuration template for the verification messages sent to a newly registered user. Supports email and phone message templates which basically contains a code. If `self_sign_up_enabled` is set to False, then users can only be verified by app admins by [`admin-confirm-sign-up`](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/admin-confirm-sign-up.html)
- [`standard_attributes`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/StandardAttributes.html): List of pre-defined attributes associated with a user.
- [`custom_attributes`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/CustomAttributeProps.html): Map of custom attributes associated with a user. Cognito currently supports 50 custom attribute definitions which cannot be either deleted or updated. This is the part I don't really like about Cognito. You can access more information [here](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-settings-attributes.html)
- [`password_policy`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/PasswordPolicy.html): A set of rules for defining the password policy.
- [`account_recovery`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/AccountRecovery.html): Defines the method for recovering an account. It can only be either email or phone.
- [`removal_policy`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.RemovalPolicy.html): Property for specifying if UserPool should be deleted or retained. The default behaviour for UserPool is `RETAIN` which means even we destroy the stack UserPool will remain in orphan state.

Additionally, here are the few properties we used for creating an AppClient;
- `user_pool_name`: Name of the AppClient. Again, naming aws resources is not a good practice
- [`auth_flows`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/AuthFlow.html): Types of the auth flows AppClient will support. In this example we are just using simple username password flow.

<br/>

## Deploying The CDK Stack
> cdk deploy CognitoUserPoolStack

After a successful deployment you will get an output like;
```
Synthesis time: 1.89s

CognitoUserPoolStack: deploying...
[0%] start: Publishing 7c7606651f89dfb939de56c0ad9c7993230f8f456f5a0280bd31e5819afd8f2a:us-east-1
[100%] success: Published 7c7606651f89dfb939de56c0ad9c7993230f8f456f5a0280bd31e5819afd8f2a:us-east-1
CognitoUserPoolStack: creating CloudFormation changeset...

CognitoUserPoolStack

Deployment time: 28.64s

Stack ARN:
arn:aws:cloudformation:us-east-1:stack/CognitoUserPoolStack/3b368ac0-c0d5-11ec-96d8-127bd92f1131

Total time: 30.53s
```

<br/>

## Creating a User From AWS CLI
There are a bunch of steps for registering and confirming a new user in cognito. We will use [cognito-idp](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/index.html) commands. Also, I will use dummy email account getting one from https://temp-mail.org/en

### Sign-up User
The first step is sign-up a user by [sign-up](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/sign-up.html) command.

> aws cognito-idp sign-up \\\
--client-id \<app_client_id\> \\\
--username wiyovah293@carsik.com \\\
--password 1234Abcd^ \\\
--user-attributes Name="email",Value="wiyovah293@carsik.com" Name="family_name",Value="Foobar" Name="custom:tenant_id",Value="1234567890" Name="custom:created_at",Value="2022-01-01" Name="custom:employee_id",Value="10" Name="custom:is_admin",Value="false" \\\
--region us-east-1

Response;
```json
{
  "UserConfirmed": false,
  "CodeDeliveryDetails": {
    "Destination": "w***@c***",
    "DeliveryMedium": "EMAIL",
    "AttributeName": "email"
  },
  "UserSub": "38f58e04-96a4-4ab0-94c4-2d9ce5bd0ab1"
}
```

<br/>

### Confirm User
Next step is confirming the user by [admin-confirm-sign-up](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/admin-confirm-sign-up.html) command. Since this is an admin command we don't have to provide the verification code. This will move user's confirmation state from `Unconfirmed` to `Confirmed` state.

> aws cognito-idp admin-confirm-sign-up \\\
--user-pool-id \<user_pool_id\> \\\
--username wiyovah293@carsik.com \\\
--region us-east-1

<br/>

### Update User Attribute
Final step is updating user's email attribute as verified by calling the command [admin-update-user-attributes](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/admin-update-user-attributes.html)

> aws cognito-idp admin-update-user-attributes \\\
--user-pool-id \<user_pool_id\> \\\
--username wiyovah293@carsik.com \\\
--user-attributes Name=email_verified,Value=true \\\
--region us-east-1

<br/>

## Creating a User From Web App
We will create a user from a web app which uses AWS Amplify. The related project can be from my [github](https://github.com/flexelem/react-amplify-simple-login-form) repository. It only
has two input fields for taking user email and password. The standard and custom attributes are given as hard coded parameters for signup. Also, the two methods we are calling can be found
at Amplify [documentation](https://docs.amplify.aws/lib/auth/emailpassword/q/platform/js/).

Here is the simple Amplify signup function.
```javascript
const { user } = await Auth.signUp({
  username: email,
  password,
  attributes: {
    email: email,
    family_name: 'foobar',
    'custom:tenant_id': '1234567890',
    'custom:created_at': '2022-01-01',
    'custom:employee_id': '10',
    'custom:is_admin': false,
  },
});
```

<br/>

After a successful response we will receive a verification email based on the template we defined.

![verify_email]({{ site.url }}/public/images/2022/04/verify_email.png)

And the second part is confirming signup of the user.
```javascript
const result = await Auth.confirmSignUp(this.state.email, verifyCode);
```

<br/>

Finally, our cognito user pool will have a verified user

![verified_user]({{ site.url }}/public/images/2022/04/cognito_verified_user.png)

<br/>

## User Signin
I will use another AWS CLI command for signing in and verify that we are getting related JWT tokens.

> aws cognito-idp initiate-auth --auth-flow USER_PASSWORD_AUTH \\\
--client-id \<app_client_id\> \\\
--auth-parameters USERNAME=wiyovah293@carsik.com,PASSWORD=1234Abcd^ \\\
--region us-east-1

```json
{
    "ChallengeParameters": {},
    "AuthenticationResult": {
        "AccessToken": "eyJraWQiOiJpbXZXXC9haUM3UlhcL2J5ajMxbjFMc21SOEVoV3RrYU15NE1hWklVczFBdmM9IiwiYWxnIjoiUlMyNTYifQ.eyJvcmlnaW5fanRpIjoiNTdiNjkzYTctNjc4Yi00NGU4LTgxZmItNjI5OWUwNGI2YzkwIiwic3ViIjoiMGQ0NDQxNTUtMTBjMy00NGM2LWFlN2EtODkxNGQ1NDRkY2I3IiwiZXZlbnRfaWQiOiI3YTgwNWEyNy1mZTZjLTRkY2QtODhiYy1lNmJkNTZmM2U2ZTQiLCJ0b2tlbl91c2UiOiJhY2Nlc3MiLCJzY29wZSI6ImF3cy5jb2duaXRvLnNpZ25pbi51c2VyLmFkbWluIiwiYXV0aF90aW1lIjoxNjUwNTQ0OTY1LCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9vOWpOQUlKZTAiLCJleHAiOjE2NTA1NDg1NjUsImlhdCI6MTY1MDU0NDk2NSwianRpIjoiNmU3MjJlMWEtZjU2NS00ZTk2LTlhN2ItOGE2NTVlOTA1ZTk5IiwiY2xpZW50X2lkIjoiNHYxNDBmNnAwYmdoc2F1OGRiZzg4OWJrbmMiLCJ1c2VybmFtZSI6IjBkNDQ0MTU1LTEwYzMtNDRjNi1hZTdhLTg5MTRkNTQ0ZGNiNyJ9.VbBHGl1QV7AZbx5Nt8zL29H0NCl81B_zONnlmGBNo9qxSkbzoNghscudJxvqgXOvz-HP3v6g0hzznSwL1RAMohxf5u3ebb6EOWwqh6v6BRQIIui2-BpCS4VQe3RIQTb2DbUyaoj6Rw2CUOUqXY9_LYHHtSKEplo7hvJbjky6BhCnqGySbBGgVRlA44Ad-7ek4IT3fVxwS3PRbSFEik4tRCBCD9aPCClePCszWFIQsPCc74vBvgHnA9Nfx8CgFl2Sp9J6LA9HcZ8n2gfzQpm_8w5uFIMvr40nBFxyVgSnnFzNBKlVi7ymEH0u30bDpuMfxqgWf0PcvHL1kOnLSVUSAw",
        "ExpiresIn": 3600,
        "TokenType": "Bearer",
        "RefreshToken": "eyJjdHkiOiJKV1QiLCJlbmMiOiJBMjU2R0NNIiwiYWxnIjoiUlNBLU9BRVAifQ.C2IhmkY56RENu3SJVfFElRvM3FzLPe1ZE2cIHh5AhQ2XVlWv2n1sBqeamNdTiU9z3UCHPaTpZNojM2TOoF5iCeL3cU8CKQ5gT-Zg3_TvcvKkacbY2Z6SCiHhSKNO0FpTTBogRD2B4emZ1b_CrYonYxvD53z_-O6Be1PfySfVZ0n3Ma5C1BS3Xkh5VTStvmDQXMpOeyEmmdMBIegUCqspjLyPuH_MZYe8myWIn6MScSyBXW2NqCCYqAzmpeonm3rCLcPoj2yX_-l62ipmDp_vg9DXmZElCXJ9jwWSTZx2xs4rZ4KXrI9VzOD2YffHo4s6t8H_xRyAj8a9eGiTKfKpRQ.dQxIskelYmevgO4g.CHlSryfiqhQUSvnwOXG5SlmlTueWOHBAppqXG7UKX-ftSnj1WLiO5C0akdpOQE8yA6UXl4ILBNemiBrzzuQejRbwlOcGAwAiVgFFCHv0KjHIa_IQi-xYFw3lQAOMVY7LGZAetZ8YmRgPqmDIv6xZ6WdOtDjnDaFQ0P7CGA2DbzFRJU3nKFONG_96sfLg3h5J2qW6LduVKGv49wd1eWhR_ow2p07OkUTbi0Xsnik1Tf7_5v95F9jTUnjazihSvf_s6yOg8ENNWY7fAnbkDFdyymJ5gTZyn6xXH2ohM1zYuAjlDsAQRpisltlIMABoXrvRfVP7vC6sv_uHreM3HisbRJd5MI92ge8trrg5pckCE9Yb9LsRtsU_Soq64tdBb1CLtGGyDNaiv1oJNrUIrthq_1FTgR_z25HK48TgGbqqWIsKI1g45GGcHKZF5_pGb2rzLfVRgNaUvJmdZki_kSyiAkWmwhBb8nRnvCVzk8R7fc-Q4L14i5KrGCKBp-u91X8JV-1ZtGTAB9x7RIjtEcpLaVt5wrlp9oyExgBLB6ip5xmESoOtdsTA_gwdmBdR-bGYQTt88PXSrTvtqlC2iTbKQWL6G7fYx44VVLgh5sAcP2Pr-mnUQJ5F9cLnZdh9ky95ce2xPuZxodDRkpqc45WQSBdAxgo6l0Dc-THKRb_4ghULInSTui5Yd04uM_A7tuJfR-UUjqQ4daaG7deZbtNXeswbzWqjkJ8okpluKfxDg8IYfZW_1t1MXS_IM7_WDT03wT_4MyOAxpw9jiUBhq7lRnrEs6mhNP4W_Otca4BbpoR0YPX5KBJJKYokYU5rguCscl-359vLNcr-4Yk96lFT0Nou_wge192tMS8IIy-q3_No5bPXp847oappidVtL_kUw7TiklZnb2AeAWRREni0eDWIWYRBNvVs3zU4qvgM1sgOUsekorE5nGin3pLUn3jd7h2oifAXY7g77Z-o-5z9q6DfMNr8M1m8NmdKYQc9wy5JDVI4eCwWeMArxs61xYOIfkfxflxL5U3szGYeXtJulBLdVla03oU9DoPPUif8n2kdOItyPjRTohI0WaMya_2NTE9rKd-IYYvEboiM13m6msWy1FjObllo1yXdObG5Efp3jHeOK1V3BluL2kEasChnuD0NhfYgLNcHdbbwxUMuE_YbCNrKkbzH14OwyicHSbdznz_OE2pM54b-lDmEkp2aXVoPsdUOC6o4JgFA_94D6zPbQlRMlD-JTAHwnvCBYtbz4w3Wyxfasz7YCufm0R3YLJp4kd53An5DhXi9siENp0Pd76VC0VqaHAbqeXtmluKAySQX2j-aKgIkcQ.7BRGLWY8Y-nWrNLuUNC3bA",
        "IdToken": "eyJraWQiOiJzaEc0dkdiNVwva3JlREtuUXhpT3h5bU1lWkozVkVnS3J0Mkh3R1wvaGNXS1U9IiwiYWxnIjoiUlMyNTYifQ.eyJjdXN0b206Y3JlYXRlZF9hdCI6IjIwMjItMDEtMDEiLCJzdWIiOiIwZDQ0NDE1NS0xMGMzLTQ0YzYtYWU3YS04OTE0ZDU0NGRjYjciLCJlbWFpbF92ZXJpZmllZCI6dHJ1ZSwiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLWVhc3QtMS5hbWF6b25hd3MuY29tXC91cy1lYXN0LTFfbzlqTkFJSmUwIiwiY29nbml0bzp1c2VybmFtZSI6IjBkNDQ0MTU1LTEwYzMtNDRjNi1hZTdhLTg5MTRkNTQ0ZGNiNyIsImN1c3RvbTp0ZW5hbnRfaWQiOiIxMjM0NTY3ODkwIiwiY3VzdG9tOmVtcGxveWVlX2lkIjoiMTAiLCJvcmlnaW5fanRpIjoiNTdiNjkzYTctNjc4Yi00NGU4LTgxZmItNjI5OWUwNGI2YzkwIiwiYXVkIjoiNHYxNDBmNnAwYmdoc2F1OGRiZzg4OWJrbmMiLCJldmVudF9pZCI6IjdhODA1YTI3LWZlNmMtNGRjZC04OGJjLWU2YmQ1NmYzZTZlNCIsInRva2VuX3VzZSI6ImlkIiwiYXV0aF90aW1lIjoxNjUwNTQ0OTY1LCJleHAiOjE2NTA1NDg1NjUsImlhdCI6MTY1MDU0NDk2NSwiZmFtaWx5X25hbWUiOiJmb29iYXIiLCJqdGkiOiI4ZTQ3NTA4OS0wNDk3LTQ1NzItYTc0My0xYzg5ODkxNDNhNGQiLCJlbWFpbCI6IndpeW92YWgyOTNAY2Fyc2lrLmNvbSJ9.ETxGAC4wPvnCj_YdIW4__20S7mkd1MFYbdUUMtLtU6JKaQomxLic3Jvg09Y0adx0Pc4IB3DmAs7u9D63qJVK045trFhKWrszj0znAiK1r7blhREsiYWBScUn1rql2tpup7Mz9SzU9_resQKzWTgshqvdPRuGPcqjFsKy-FE7xvBv25-JBn8tHt2XZjqmDBR_YwXeP4pAsDTezb9b3XQY_XzmOTp2_I0dOTT2O-gleF3itRDmS5FelO5C9476143e0i7zKv5MXfe3VhmcoUvZWgSJ7W_ElkEJtg7lPkP-gcj0SXeHkBF1EWjE7gJnVbNPEvJUyKBlyXYF3AyaQSZXGw"
    }
}
```

<br/>

## Destroy the Stack
Don't forget to delete the stack after your testing.

> cdk destroy CognitoUserPoolStack

<br/>

Hopefully, this was a helpful guide for creating AWS Cognito UserPool by AWS CDK. Here you can also find the related github [repository](https://github.com/flexelem/cognito-userpool-cdk-example)


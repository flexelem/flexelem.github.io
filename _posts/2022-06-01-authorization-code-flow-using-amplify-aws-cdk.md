---
title:  OAuth Authorization Code Flow with AWS Cognito in AWS CDK
author: buraktas
layout: post
permalink: /oauth-authorization-code-flow-aws-cdk/
dsq_needs_sync:
- 1
categories:
  - aws
  - aws-cdk
  - authorization
  - oauth
tags:
  - aws-cdk
  - aws
  - aws-cognito
  - aws-amplify
  - aws-apigateway
comments: true
---
We created a Cognito User Pool and demonstrated a simple authentication by registering and logging in a new user in my [previous]({{ site.url }}/create-cognito-user-pool-aws-cdk/) tutorial. Now we will take a step further by adding a common OAuth
authorization step which is [`OAuth Authorization Code Flow`](https://datatracker.ietf.org/doc/html/rfc6749#section-4.1) with a super simple web app. With this example we will see how our web app can call protected APIs. These are the resources we will provision;

- Create a AWS Cognito User Pool
- Create a AWS Cognito App Client with Authorization Code Flow
- Create a Resource Server (with a custom Cognito Domain)
- Create a protected API from API Gateway
- Verify that authenticated user is able to call the protected API with provided jwt tokens.

<!--more-->

Now before deep diving into examples lets first understand how OAuth Authorization Code Flow works which is a redirection based flow. First of all, OAuth 2.0 (Open Authorization) is an authorization protocol for accessing secured resources. After a user
is authorized an access token is being issued generally in JWT format. This access token has an expiry date which is configurable. So, after the token is expired user has to obtain a new access token to continue calling protected resources. Moreover,
each protected resource may also require `OAuth 2.0 Scopes`. Scopes are a way to limit access for an access token. For example; some access tokens may be granted read and write access for protected resource, on the other hand, some will only have read access. Authorization Code
Flow is an OAuth 2.0 flow based on a redirection (redirect URI) from the Authorization server to exchange returned `code` with an access token.

![auth_code_flow]({{ site.url }}/public/images/2022/06/auth_code_flow.png)

- User clicks on Login page on the Web Client.
- Web Client makes an [`/authorize`](https://docs.aws.amazon.com/cognito/latest/developerguide/authorization-endpoint.html) request to initiate Authorization Code Flow. This is a GET request including `client_id`, `redirect_uri`, `response_type` and `scope` (optional) query parameters.
- Authorization Server returns a redirect response with a [`/login`](https://docs.aws.amazon.com/cognito/latest/developerguide/login-endpoint.html) endpoint where the user will be redirected to enter their credentials.
- User enters their credentials through a Hosted UI.
- Authorization Server redirects user back to given `redirect_uri` after a successful login with an url containing a `code` query parameter. This `code` will be used
  later on for exchanging it with an access token.
- Web Client will make a [`/token`](https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html) endpoint request by including the returned `code` to get an access token.
- Auth Server will return an access token (along with id token and refresh token if configured).
- User will be able to call protected APIs by using given access token.

<br/>

Here is the same representation of Authorization Code Flow by using AWS architecture including different resources;

![aws_auth_code_flow]({{ site.url }}/public/images/2022/06/aws_auth_code_flow.png)

- User clicks on the login page which makes an `/authorize` call by the browser.
- AWS Cognito will return a redirect response for a login url and user will be asked for entering their credentials.
- After a successful login, Cognito Hosted UI will redirect user back to Web Client with a URL containing a `code` parameter.
- AWS Amplify will handle this redirection behind the scenes and will make a `/token` call to exchange the returned `code` parameter with a valid access token.
- AWS Cognito will return a valid access token (along with id and refresh tokens which are optional)
- User can call protected resources with returned access token.
- Once API Gateway receive the request it will pass the access token and scopes to AWS Cognito for checking their validity.
- AWS Cognito will confirm if the tokens and scopes are valid.
- Finally, API Gateway will return a success response back to Web Client.

<br/>

## Login Flow

We will demonstrate a login flow with two different ways which will be from a web client and by making http requests with curl commands.

### 1) Web Client

I will go through with a very simple react app I implemented only for this project. You can find the related repository from [here](https://github.com/flexelem/aws-cognito-hosted-ui-react)

Well as like any other web app we have a landing page containing only one Login button. Once, a user clicks on it they will be redirected to the Cognito Hosted UI. This URL is hardcoded which you can obtain from AWS Hosted UI Console and looks like

> https://buraktas-awesome-domain.auth.us-east-1.amazoncognito.com/oauth2/authorize?client_id=2v4929s76jtnepd4q0nj695gmb&response_type=code&scope=awesomeapi-resource-server%2Fawesomeapi.read+openid&redirect_uri=http%3A%2F%2Flocalhost%3A3000%2Fdashboard


<br/>


![landing_page]({{ site.url }}/public/images/2022/06/landing_page.png)

<br/>

After clicking on login button browser makes an `/authorize` request and Cognito returns a redirect response with a URL (in location header) pointing to a login page from their Hosted UI.

![browser_authorize_request]({{ site.url }}/public/images/2022/06/browser_authorize_request.png)


<br/>

Browser will redirect user to the login page which includes query parameters;
- client_id
- response_type
- scope
- redirect_uri

![browser_redirection_login]({{ site.url }}/public/images/2022/06/browser_redirection_login.png)

<br/>


This is the Cognito Hosted UI login page. User will be redirected back to the `redirect_uri` provided after they enter their credentials with a `code` query parameter.

![aws_cognito_hosted_ui]({{ site.url }}/public/images/2022/06/aws_cognito_hosted_ui.png)

<br/>

AWS Cognito returns a redirect response.

![redirect_back_web_client]({{ site.url }}/public/images/2022/06/redirect_back_web_client.png)

<br/>

AWS Amplify will make `/token` request behind the scenes after the user is redirected back to the web client. Later on, user will get a valid access token.

![token_request]({{ site.url }}/public/images/2022/06/token_request.png)

<br/>

![token_request_form_data]({{ site.url }}/public/images/2022/06/token_request_form_data.png)

<br/>

User will get the valid tokens

```
{
  "id_token": "eyJraWQiOiJNd3Q...",
  "access_token": "eyJraWQiOi...",
  "refresh_token": "eyJjdHkiOi...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

<br/>

As the final step, user is able to call protected APIs. Here is the super simple Dashboard page which contains a button to make a call to our protected API. Once we click on it we will see a success response.

![redirected_to_redirect_uri_web_client]({{ site.url }}/public/images/2022/06/redirected_to_redirect_uri_web_client.png)

<br/>

### 2) Curl Commands

We can assume user enters their credentials through a browser and redirected back to the web client with a URL like;

`http://localhost:3000/dashboard?code=426e230c-ad12-4112-ab0e-9685e8ef9ac9`

Here is the Curl request to the `/token` endpoint we use;

```
curl --location --request POST 'https://buraktas-awesome-domain.auth.us-east-1.amazoncognito.com/oauth2/token' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'client_id=2v4929s76jtnepd4q0nj695gmb' \
--data-urlencode 'code=1169af5a-c718-45cb-8beb-ca05e5509550' \
--data-urlencode 'grant_type=authorization_code' \
--data-urlencode 'redirect_uri=http://localhost:3000/dashboard'
```

And here below is the response with a short format. We can see main the attributes `scope`, `client_id` and `username` if we decode the access token.

```
{
	"access_token": "eyJraWQiOiJyOVBsUVF1SUtsMlwvcEt2ZXhlWjQ5dVlMXC81THRxR2o5UElFb3o3MEJueUk9IiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiIyN2I4MDBiYy00NDdkLTRhY2ItYTIwNi0zYmYzNTY4MzZmZmMiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9QdjhQcG01elEiLCJ2ZXJzaW9uIjoyLCJjbGllbnRfaWQiOiIydjQ5MjlzNzZqdG5lcGQ0cTBuajY5NWdtYiIsIm9yaWdpbl9qdGkiOiI2MWJjZjIyZC1mYmEyLTQyYzgtYTg2MS1lMTE0NjQxYmIwODMiLCJldmVudF9pZCI6ImUxNjQxM2EzLTJjOTctNDYwMS05ZDJkLTc3MGY1NjRjNWZjMiIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdlc29tZWFwaS1yZXNvdXJjZS1zZXJ2ZXJcL2F3ZXNvbWVhcGkucmVhZCBvcGVuaWQiLCJhdXRoX3RpbWUiOjE2NTQwODUxMzEsImV4cCI6MTY1NDA4ODczMSwiaWF0IjoxNjU0MDg1MTMxLCJqdGkiOiI5ZDk4YzY5Ni1mYjMxLTQ0YzQtYmRlZi1iNmZmMzFjMGUxOTEiLCJ1c2VybmFtZSI6IjI3YjgwMGJjLTQ0N2QtNGFjYi1hMjA2LTNiZjM1NjgzNmZmYyJ9.k8Go2FUcObywizvrVUfdU0rYSyFyNJgAN-rwjL6WTzgocc5vm3giu2hRfhfS57iz9Z59XevO5vzF5xnuLXiz156WgwR6U5Yo8ku-ZTLhEzXHN1wXy82VLoFaKZXJod9fjpI0vCCoNpsRKHGzteaHI5PRN7r1td0aHgKZX8VKwZovsLQueEqwHQGh1f6FWXaygvm_uRF5X43dNUAy_j8n4gv9X4hMm7CKJSS4bm0MzeptV7Z1eCD3sCuWANq4SCHpXL4TvpROpoF26Tt9ZaGoQy6xUiM0K-v2TWU1kTYcQpXtXlq3UDrE4RV3xG4R3lg5G5HwzCgxwlAbz-8IpJl-Mg",
	"id_token": "eyJraWQiOiJNd3Q...",
	"refresh_token": "eyJjdHkiOiJ...",
	"expires_in": 3600,
	"token_type": "Bearer"
}
```

Now user is able to call protected APIs. We have a GET API which requires a scope of `awesomeapi-resource-server/awesomeapi.read`. Of course, this scope doesn't come by default. It is the way that I configured the Resource Server for granting access with this scope and
 provisioned an API which checks if the access token has it (Remember the step API Gateway checks the validity of the access token).

```
curl --request GET 'https://u9l0thnf46.execute-api.us-east-1.amazonaws.com/prod/awesomeapi' \
--header 'Authorization: eyJraWQiOiJyOVBsUVF1SUtsMlwvcEt2ZXhlWjQ5dVlMXC81THRxR2o5UElFb3o3MEJueUk9IiwiYWxnIjoiUlMyNTYifQ.eyJzdWIiOiIyN2I4MDBiYy00NDdkLTRhY2ItYTIwNi0zYmYzNTY4MzZmZmMiLCJpc3MiOiJodHRwczpcL1wvY29nbml0by1pZHAudXMtZWFzdC0xLmFtYXpvbmF3cy5jb21cL3VzLWVhc3QtMV9QdjhQcG01elEiLCJ2ZXJzaW9uIjoyLCJjbGllbnRfaWQiOiIydjQ5MjlzNzZqdG5lcGQ0cTBuajY5NWdtYiIsIm9yaWdpbl9qdGkiOiI4MDFiMjAwZC1lYzljLTQ3ZmMtYmMzOS0yN2Q5ZjAyOWFjNjUiLCJldmVudF9pZCI6IjhmZGMyZDdiLWVkMTUtNGI0Yi04NDc1LWQ5NTE1MDAzZDUzNCIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdlc29tZWFwaS1yZXNvdXJjZS1zZXJ2ZXJcL2F3ZXNvbWVhcGkucmVhZCBvcGVuaWQiLCJhdXRoX3RpbWUiOjE2NTQxMDQ0NTYsImV4cCI6MTY1NDEwODA1NiwiaWF0IjoxNjU0MTA0NDU3LCJqdGkiOiI4MjY1ZjhhOC1jMGYyLTRiZjctODYyNS1iMjIyMjA5OTM3NDciLCJ1c2VybmFtZSI6IjI3YjgwMGJjLTQ0N2QtNGFjYi1hMjA2LTNiZjM1NjgzNmZmYyJ9.BmBLqhShOiuhD8Q2zAVK0M3qtBENXVzgiCpfw5W839M6oFBcWkNCaTnUJubujhu4FpUHDVssDY1CTW0CbJS4ULrj1vcnwZSALF-AcNQjxyMC2xNhqCo_r92fGm66txnUZzWftdN_Tg62mgEprflZ0tVR-PU9cbAI-DDko5fUUANkR1CSDQf0A36cW3ttkb4N0PxyAWHmKezn7dX62l1uha4THuWci8MB2t7G06u5SM2f2azOdFlCAy5Ai-fgvlJK_jYqPBPY-K__N3tjVJq9DfIEWtgHnOhwCPsAhByBZdFSEuuLTGFtWrhCcv9m6DasGc76h03coBTQPFSv2raAcA'
```

Response
```json
{
    "statusCode": 200,
    "message": "Hello From Protected Resource"
}
```

<br/>

## CDK Code
We will have two different CDK Stacks for AWS Cognito and AWS ApiGateway.


### AWS Cognito Stack

```python
from aws_cdk import (
    core,
    aws_cognito as cognito,
)


class CognitoStack(core.Stack):
    def __init__(self, scope: core.Construct,
                 id: str,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        self.cognito_user_pool = cognito.UserPool(self, "awesome-cognito-user-pool",
            sign_in_aliases=cognito.SignInAliases(
                email=True
            ),
            self_sign_up_enabled=True,
            auto_verify=cognito.AutoVerifiedAttrs(
                email=True
            ),
            user_verification=cognito.UserVerificationConfig(
                email_subject="You need to verify your email",
                email_body="Thanks for signing up Your verification code is {####}",
                email_style=cognito.VerificationEmailStyle.CODE
            ),
            account_recovery=cognito.AccountRecovery.EMAIL_ONLY,
            removal_policy=core.RemovalPolicy.DESTROY
        )

        awesome_api_read_scope = cognito.ResourceServerScope(scope_name="awesomeapi.read", scope_description="awesomeapi read scope")

        resource_server = cognito.UserPoolResourceServer(
            self,
            "awesome-resource-server",
            identifier="awesomeapi-resource-server",
            user_pool=self.cognito_user_pool,
            scopes=[awesome_api_read_scope]
        )

        user_pool_app_client = cognito.UserPoolClient(
            self,
            "awesome-app-client",
            user_pool=self.cognito_user_pool,
            access_token_validity=core.Duration.minutes(60),
            id_token_validity=core.Duration.minutes(60),
            generate_secret=False,
            refresh_token_validity=core.Duration.days(1),
            enable_token_revocation=True,
            prevent_user_existence_errors=True,
            auth_flows=cognito.AuthFlow(
                user_password=True,
            ),
            o_auth=cognito.OAuthSettings(
                flows=cognito.OAuthFlows(
                    authorization_code_grant=True,
                ),
                scopes=[cognito.OAuthScope.OPENID,
                        cognito.OAuthScope.resource_server(resource_server, awesome_api_read_scope)
                ],
                callback_urls=["http://localhost:3000/dashboard"],
            )
        )

        self.cognito_user_pool.add_domain("awesome-cognito-domain",
            cognito_domain=cognito.CognitoDomainOptions(
                domain_prefix="buraktas-awesome-domain"
            )
        )

        core.CfnOutput(self, "CognitoUserPoolID", value=self.cognito_user_pool.user_pool_id)
        core.CfnOutput(self, "CognitoUserPoolAppClientID", value=user_pool_app_client.user_pool_client_id)
```

<br/>

I will skip the `cognito.UserPool` construct which is the same and a short version of the previous tutorial.

**cognito.ResourceServer**
- `identifier`: This is just an identifier for the resource server. (It is not the id of the CDK construct)
- `user_pool`: Related Cognito UserPool to attach this Resource Server
- `scopes`: The list of available scopes we want to grant access for App Clients. Remember that we were authorized to call our protected API with the given scope of `awesomeapi-resource-server/awesomeapi.read`.
So, it has a format like {resource_server_identifier/scope_name}


<br/>


**cognito.UserPoolClient**
- `user_pool`: The associated UserPool this AppClient will use.
- `access_token_validity`: Validity of the access token.
- `id_token_validity`: Validity of the id token.
- `refresh_token_validity`: Validity of the refresh token.
- `prevent_user_existence_errors`: To make Cognito return more abstract errors instead of indicating that user doesn't really exist, invalid password etc.
- `generate_secret`: Whether to generate a client secret for the App Client. In our case, we don't need this secret since we are using this App Client on Web App and we don't want to expose it to public.
- [`auth_flows`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/AuthFlow.html): Types of the auth flows AppClient will support. In this example we are just using simple username password flow.
- [`o_auth`](https://docs.aws.amazon.com/cdk/api/v2/python/aws_cdk.aws_cognito/OAuthSettings.html#aws_cdk.aws_cognito.OAuthSettings): OAuth settings including flow type, scopes and callback urls. For this tutorial, we
are using `authorization_code_grant` OAuth flow.

NOTE: There is also another scope `OPENID` which is not related to this example, however, due to a bug from AWS Amplify we have to include this scope as well. Otherwise, AWS Amplify will return an error
when using `Auth.currentAuthenticatedUser()` after the user is redirected back to the provided `redirect_uri`. In our case, it is the Dashboard page (which is also a protected UI page). Here is the related [Github Issue](https://github.com/aws-amplify/amplify-js/issues/9920)

As the last step, we also create a Cognito Domain with a custom name. And attach it to Cognito UserPool


<br/>

### AWS APIGateway Stack

In this stack we are just creating a RestApi with one protected GET endpoint. We will not deep dive into this stack. It is basically a [Mocked Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-mock-integration.html) to support
basic request response model. We will get a custom API URL after the deployment. The only important part here is configuring a `CognitoUserPoolsAuthorizer` and defining which authorization scopes are required for the GET method.

```python
import json
from aws_cdk import (
    core,
    aws_cognito as cognito,
    aws_apigateway as api_gateway
)


class ApiGatewayStack(core.Stack):
    def __init__(self, scope: core.Construct,
                 id: str,
                 cognito_user_pool: cognito.IUserPool,
                 **kwargs) -> None:
        super().__init__(scope, id, **kwargs)

        awesome_api = api_gateway.RestApi(
            self,
            "awesome-api",
            endpoint_types=[api_gateway.EndpointType.REGIONAL],
            deploy=True,
            deploy_options=api_gateway.StageOptions(
                stage_name="prod",
            ),
            default_cors_preflight_options=api_gateway.CorsOptions(
                allow_origins=api_gateway.Cors.ALL_ORIGINS,
                allow_methods=api_gateway.Cors.ALL_METHODS,
            )
        )

        cognito_userpool_authorizer = api_gateway.CognitoUserPoolsAuthorizer(
            self,
            "cognito-userpool-authorizer",
            cognito_user_pools=[cognito_user_pool]
        )

        # /awesomeapi
        awesome_api_resource = awesome_api.root.add_resource("awesomeapi")

        awesome_api_resource.add_method(
            "GET",
            api_gateway.MockIntegration(
                integration_responses=[api_gateway.IntegrationResponse(
                    status_code="200",
                    response_templates={
                        "application/json": json.dumps({
                            "statusCode": 200,
                            "message": "Hello From Protected Resource",
                        })
                    },
                    response_parameters={
                        "method.response.header.Content-Type": "'application/json'",
                        "method.response.header.Access-Control-Allow-Origin": "'*'"
                    }
                )],
                request_templates={
                    "application/json": "{ 'statusCode': 200 }"
                }
            ),
            method_responses=[api_gateway.MethodResponse(
                status_code="200",
                response_parameters={
                    "method.response.header.Content-Type": True,
                    "method.response.header.Access-Control-Allow-Origin": True
                }
            )],
            authorizer=cognito_userpool_authorizer,
            authorization_type=api_gateway.AuthorizationType.COGNITO,
            authorization_scopes=["awesomeapi-resource-server/awesomeapi.read"]
        )
```

<br/>

## Synth Stack(s)
> cdk synth

```
Supply a stack id (AuthCodeFlowStack, AuthCodeFlowStack/CognitoStack, AuthCodeFlowStack/ApiGatewayStack) to display its template.
```

<br/>

## Deploy Stack(s)
> cdk deploy --all

<br/>

## Destroy the Stack
Don't forget to delete the stack after your testing.

> cdk destroy --all


<br/>

I hope you enjoyed my tutorial about OAuth Authorization Code Flow in AWS CDK. Here you can also find the related github [repository](https://github.com/flexelem/authorization-code-flow-aws-cdk). Please don't hesitate any questions if you have.

---
title:  OAuth 2.0 Client Credentials Flow with AWS Cognito in AWS CDK
author: buraktas
layout: post
permalink: /oauth-client-credentials-flow-aws-cdk/
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
  - aws-apigateway
comments: true
---

We learned how to implement OAuth Authorization Code Flow which provides client facing apps to access protected resources in my [previous]({{ site.url }}/oauth-authorization-code-flow-aws-cdk/) tutorial. Now what if there are external companies, clients, devices etc.
wants to integrate with our resources? We generally have been provided some sort of credentials which can be a basic username password tuple, api key, even a unique string to make a call to 3rd party clients.
In this example we will learn [`Oauth Client Credentials Flow`](https://datatracker.ietf.org/doc/html/rfc6749#section-4.4).
This flow is being used for Machine-to-Machine (M2M) communication. Basically, the client has to get an access token for making calls to protected endpoints. Similar to the other OAuth flows, these protected endpoints might require different scopes from each other as well.
These are the resources we will provision;

- Create a AWS Cognito User Pool
- Create a AWS Cognito App Client with Client Credentials Flow
- Create a Resource Server (with a custom Cognito Domain)
- Create a protected API from API Gateway
- Verify that authenticated user is able to call the protected API with provided jwt tokens.

<!--more-->

Client credentials flow is a simple which contains a few steps to get an access token to provide M2M communication.

![client_credentials_flow]({{ site.url }}/assets/img/2022/06/client_credentials_flow.png)

- Server app makes a call to [`/token`](https://docs.aws.amazon.com/cognito/latest/developerguide/token-endpoint.html) endpoint with Client ID and Client Secret pair to request access token. `Authorization` request header is mandatory which is
in format of **Base64Encode(client_id:client_secret)**. Additionally, in request body `grant_type` parameter must be `client_credentials` and `scope` should be provided if there any scopes associated to the app client.
- Auth Server returns a valid access token after validating the provided credentials.
- Server app will be able to call protected APIs by using access token.

<br/>

Here is the AWS representation of the Client Credentials Flow;

![aws_client_credentials_flow]({{ site.url }}/assets/img/2022/06/aws_client_credentials_flow.png)

- Server app makes a call `/token` endpoint with providing Client ID and Client Secret pair to get an access token.
- AWS Cognito validates provided Client ID and Client Secret pair. Returns access token after if the credentials are valid.
- Server app can call protected APIs with the access token.
- Once API Gateway receive the request it will pass the access token and scopes to AWS Cognito for checking their validity.
- AWS Cognito will confirm if the tokens and scopes are valid.
- Finally, API Gateway will return a success response back to Web Client.

<br/>

## Testing
First thing first, lets see we really get an `401 - Unauthorized` response from the protected endpoint by making a http call without an `Authorization` header.

```shell
> curl -i --request GET 'https://1w1wa554q4.execute-api.us-east-1.amazonaws.com/prod/awesomeapi'

HTTP/2 401
date: Mon, 06 Jun 2022 21:16:15 GMT
content-type: application/json
content-length: 26
x-amzn-requestid: 0e5fb841-7398-40ef-891c-4547327ad029
x-amzn-errortype: UnauthorizedException
x-amz-apigw-id: TUY44EBvoAMFbUw=

{"message":"Unauthorized"}
```

Now, we need to get an access token.

```shell
> curl --request POST 'https://buraktas-awesome-domain.auth.us-east-1.amazoncognito.com/oauth2/token' \
--header 'Authorization: Basic NGYyaG1obmh2anVqam9yMGtpbGE4ZThpdTk6MWhqMzVyZjE1dTNjNnUyb2FxaXV1MzUyYWprbXM0cW10bTIxNmtsN3M1ZXIwYzRhM25nYw==' \
--header 'Content-Type: application/x-www-form-urlencoded' \
--data-urlencode 'grant_type=client_credentials' \
--data-urlencode 'scope=awesomeapi-resource-server/awesomeapi.read'
```

Just to note; `scope` parameter is Optional here. If we don't request for any custom scopes we created then AWS Cognito will return an access token having all scopes defined.

```json
{
  "access_token": "eyJraWQiOiJEVGxKSTBvTnN4KzVjOFVLZDViYlJTNnl6bnFFY1UyS3VOY1l4OGc2RmNNPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiI0ZjJobWhuaHZqdWpqb3Iwa2lsYThlOGl1OSIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdlc29tZWFwaS1yZXNvdXJjZS1zZXJ2ZXJcL2F3ZXNvbWVhcGkucmVhZCIsImF1dGhfdGltZSI6MTY1NDU1NDAyNywiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLWVhc3QtMS5hbWF6b25hd3MuY29tXC91cy1lYXN0LTFfTjhGMjJVc2xvIiwiZXhwIjoxNjU0NTU3NjI3LCJpYXQiOjE2NTQ1NTQwMjcsInZlcnNpb24iOjIsImp0aSI6IjEwMzhmNTNmLTBjZTAtNGI1Zi04MDhiLTk1ZTg4MGE4NzY0MyIsImNsaWVudF9pZCI6IjRmMmhtaG5odmp1ampvcjBraWxhOGU4aXU5In0.t1qmxKwboXh4s2FcpExB_icqUkBaAn9UzR3qZPtT3_U5NuxoJ05JLHCCM9NfYUdiT9nlP08NMJSVi_qQBEwmcouWhNN9mrWQqvpuyha8_UFCrFAyzyOrjeUHsknoabyjToUPlPYbdmPP6LhjeK43lcZeJnUeXBELGIGz0mkasPbiodyvEmozAczxfikUGzStgTOXF9YueLSjs1r-JClj0QICfaZW7mMYno462fioURy-UZElVsfXODFhWIXmD9viFoEy657_sKRzctrLci0ejD9jKv_MBEBMBYiQpIEN3zyevCweXYG9jmMaGI8w-StrDGYNqdDPcn02a3kJlCV76Q",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

We are finally ready to call our protected endpoints with provided access token;

```shell
> curl --request GET 'https://1w1wa554q4.execute-api.us-east-1.amazonaws.com/prod/awesomeapi' \
--header 'Authorization: eyJraWQiOiJEVGxKSTBvTnN4KzVjOFVLZDViYlJTNnl6bnFFY1UyS3VOY1l4OGc2RmNNPSIsImFsZyI6IlJTMjU2In0.eyJzdWIiOiI0ZjJobWhuaHZqdWpqb3Iwa2lsYThlOGl1OSIsInRva2VuX3VzZSI6ImFjY2VzcyIsInNjb3BlIjoiYXdlc29tZWFwaS1yZXNvdXJjZS1zZXJ2ZXJcL2F3ZXNvbWVhcGkucmVhZCIsImF1dGhfdGltZSI6MTY1NDU1NDAyNywiaXNzIjoiaHR0cHM6XC9cL2NvZ25pdG8taWRwLnVzLWVhc3QtMS5hbWF6b25hd3MuY29tXC91cy1lYXN0LTFfTjhGMjJVc2xvIiwiZXhwIjoxNjU0NTU3NjI3LCJpYXQiOjE2NTQ1NTQwMjcsInZlcnNpb24iOjIsImp0aSI6IjEwMzhmNTNmLTBjZTAtNGI1Zi04MDhiLTk1ZTg4MGE4NzY0MyIsImNsaWVudF9pZCI6IjRmMmhtaG5odmp1ampvcjBraWxhOGU4aXU5In0.t1qmxKwboXh4s2FcpExB_icqUkBaAn9UzR3qZPtT3_U5NuxoJ05JLHCCM9NfYUdiT9nlP08NMJSVi_qQBEwmcouWhNN9mrWQqvpuyha8_UFCrFAyzyOrjeUHsknoabyjToUPlPYbdmPP6LhjeK43lcZeJnUeXBELGIGz0mkasPbiodyvEmozAczxfikUGzStgTOXF9YueLSjs1r-JClj0QICfaZW7mMYno462fioURy-UZElVsfXODFhWIXmD9viFoEy657_sKRzctrLci0ejD9jKv_MBEBMBYiQpIEN3zyevCweXYG9jmMaGI8w-StrDGYNqdDPcn02a3kJlCV76Q'
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
We will have two different CDK Stacks for AWS Cognito and AWS ApiGateway. However, since they are pretty much the same with my previous tutorial I will just pinpoint the different code I used for this tutorial.


### AWS Cognito Stack
Notice that, we created a very simple `cognito.UserPool` since we are not going to register any User. `cognito.ResourceServer` is the same as my previous tutorial with having the same scope. And
`cognito.UserPoolClient` is configured to support OAuth Client Credentials flow with the scope we defined.

```typescript
import * as cdk from 'aws-cdk-lib';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import { Construct } from 'constructs';

export class CognitoStack extends cdk.Stack {
  readonly cognitoUserPool: cognito.IUserPool;

  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    this.cognitoUserPool = new cognito.UserPool(this, 'awesome-cognito-user-pool', {
      accountRecovery: cognito.AccountRecovery.EMAIL_ONLY,
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });

    const awesomeApiReadScope = new cognito.ResourceServerScope({
      scopeName: 'awesomeapi.read',
      scopeDescription: 'awesomeapi read scope',
    });

    const resourceServer = new cognito.UserPoolResourceServer(this, 'awesome-resource-server', {
      identifier: 'awesomeapi-resource-server',
      userPool: this.cognitoUserPool,
      scopes: [awesomeApiReadScope],
    });

    const userPoolAppClient = new cognito.UserPoolClient(this, 'awesome-app-client', {
      userPool: this.cognitoUserPool,
      accessTokenValidity: cdk.Duration.minutes(60),
      generateSecret: true,
      refreshTokenValidity: cdk.Duration.days(1),
      enableTokenRevocation: true,
      oAuth: {
        flows: {
          clientCredentials: true,
        },
        scopes: [cognito.OAuthScope.resourceServer(resourceServer, awesomeApiReadScope)],
      },
    });

    this.cognitoUserPool.addDomain('awesome-cognito-domain', {
      cognitoDomain: {
        domainPrefix: 'buraktas-awesome-domain',
      },
    });
  }
}

```

<br/>

### AWS APIGateway Stack
There is nothing different from the previous tutorial except the fact that we don't need CORS. So, I removed CORS related changes in this stack. This is a API Gateway stack
containing one protected GET endpoint with MOCK integration.

```typescript
import * as cdk from 'aws-cdk-lib';
import * as apigw from 'aws-cdk-lib/aws-apigateway';
import * as cognito from 'aws-cdk-lib/aws-cognito';
import { Construct } from 'constructs';

export class ApiGatewayStack extends cdk.Stack {
  constructor(scope: Construct, id: string, cognitoUserPool: cognito.IUserPool, props?: cdk.StackProps) {
    super(scope, id, props);

    const awesomeApi = new apigw.RestApi(this, 'awesome-api', {
      endpointTypes: [apigw.EndpointType.REGIONAL],
      deploy: true,
      deployOptions: {
        stageName: 'prod',
      },
    });

    const cognitoUserPoolAuthorizer = new apigw.CognitoUserPoolsAuthorizer(this, 'cognito-userpool-authorizer', {
      cognitoUserPools: [cognitoUserPool],
    });

    // /awesomeapi
    const awesomeApiResource = awesomeApi.root.addResource('awesomeapi');
    awesomeApiResource.addMethod(
      'GET',
      new apigw.MockIntegration({
        integrationResponses: [{
          statusCode: '200',
          responseTemplates: {
            'application/json': JSON.stringify({
              statusCode: 200,
              message: 'Hello From Protected Resource',
            }),
          },
          responseParameters: {
            'method.response.header.Content-Type': "'application/json'",
          },
        }],
        requestTemplates: {
          'application/json': "{ 'statusCode': 200 }",
        },
      }), {
        methodResponses: [{
          statusCode: '200',
          responseParameters: {
            'method.response.header.Content-Type': true,
          },
        }],
        authorizer: cognitoUserPoolAuthorizer,
        authorizationType: apigw.AuthorizationType.COGNITO,
        authorizationScopes: ['awesomeapi-resource-server/awesomeapi.read'],
      },
    );
  }
}
```

<br/>

## Synth Stack(s)
```shell
> cdk synth

Supply a stack id (ClientCredentialsFlowStack, ClientCredentialsFlowStack/CognitoStack, ClientCredentialsFlowStack/ApiGatewayStack) to display its template.
```

<br/>

## Deploy Stack(s)
```shell
> cdk deploy --all
```

<br/>

## Destroy Stack(s)
Don't forget to delete the stack after your testing.

```shell
> cdk destroy --all
```

<br/>

I hope you enjoyed my tutorial about OAuth Client Credentials Flow in AWS CDK. Here you can also find the related github [repository](https://github.com/flexelem/aws-cdk-examples/tree/main/client-credentials-flow). Please don't hesitate any questions if you have.

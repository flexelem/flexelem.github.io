---
title:  API Gateway Websocket API Example with AWS CDK
author: buraktas
layout: post
permalink: /api-gateway-websocket-api-example-aws-cdk/
dsq_needs_sync:
- 1
categories:
  - aws
  - aws-cdk
  - network
tags:
  - aws-cdk
  - aws
  - aws-apigateway
  - websockets
comments: true
---

In this tutorial, we will learn how to create a basic application for publishing real-time notifications via websocket api from API Gateway. Before having API Gateway support for websockets we had to
have a separate websocket server to publish notifications or sending messages to the available connections at that point of time. API Gateway literally simplifies this process and fits it into the serverless world.
The flow basically consists of persisting the connections in a data store where we will be using `DynamoDB` and getting connections which we want to send notifications to. Websocket API has 3 routes;
- **$connect**: This route is triggered automatically when a client send a connect request.
- **$disconnect**: This route is triggered automatically when a client send a connect request.
- **$default**: Obviously this is the default handler if there are no any routes matched.

API Gateway selects routes based on the `request.body.action` value, meaning we can define custom routes to handle different type of logic. And if there is no any matching route then API Gateway will forward to request to the `$default` handler.
We won't be using `$default` handler in this tutorial since there are many examples around the web. Instead, we will see how to send real time notifications like; notify clients when an order is submitted. We can take a book seller company in Amazon
marketplace with a few online employees as an example in this tutorial. When a client orders a book belonging to this merchant all the online employees will be notified. The related flow will be like;

![websocket-flow]({{ site.url }}/assets/img/2023/04/websocket-flow.png)

<br/>

- Client sends a connect request then `$connect` handler persists the connection into DynamoDB by its `connectionId` coming from `event.requestContext.connectionId`.
- Another client calls an API which will generate a notification for other online clients.
- Client sends a disconnect request which will invoke `$disconnect` handler and the related `connectionId` will be deleted from DynamoDB. This

<br/>

## DynamoDB Schema
I will be using the simplest schema for this example where the `connectionId` will be the `Primary Key` of the table. In most cases, `connectionId` of the clients are being kept along with an entity id which can be the `Primary Key` and `connectionId`
will be the `Sort Key` so that we can get all specific online clients based on that entity id. Anyway, the REST API lambda will send a `Scan` request to get all connections.

## Lambda Handlers
We will have 3 Lambda functions where two of them belongs to Websocket API to handle `$connect`, `$disconnect` requests and one as a REST API handler.

### $connect Route Lambda Handler
When a client sends a connection request this Lambda simply gets the `connectionId` and persists into DynamoDB.

```typescript
import {
  APIGatewayProxyResultV2,
  APIGatewayProxyWebsocketEventV2,
  APIGatewayProxyWebsocketHandlerV2,
} from 'aws-lambda';
import 'source-map-support/register';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, PutCommand } from '@aws-sdk/lib-dynamodb';

const dynamoDBClient = new DynamoDBClient({
  region: process.env.AWS_REGION,
});
const dynamoDBDocClient = DynamoDBDocumentClient.from(dynamoDBClient);

export const handler: APIGatewayProxyWebsocketHandlerV2 =
  async (event: APIGatewayProxyWebsocketEventV2): Promise<APIGatewayProxyResultV2> => {
    console.log('Event:', JSON.stringify(event));

    const putCommand = new PutCommand({
      TableName: process.env.CONN_TABLE_NAME,
      Item: {
        connectionId: event.requestContext.connectionId,
      },
    });
    const resp = await dynamoDBDocClient.send(putCommand);
    console.log(`putCommand resp => ${JSON.stringify(resp)}`);

    return { statusCode: 200 };
  };
```

<br/>

### $disconnect Route Lambda Handler
It is same as `$connect` Lambda handler instead it will delete the `connectionId` from DynamoDB.

```typescript
import {
  APIGatewayProxyResultV2,
  APIGatewayProxyWebsocketEventV2,
  APIGatewayProxyWebsocketHandlerV2,
} from 'aws-lambda';
import 'source-map-support/register';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DeleteCommand, DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

const dynamoDBClient = new DynamoDBClient({
  region: process.env.AWS_REGION,
});
const dynamoDBDocClient = DynamoDBDocumentClient.from(dynamoDBClient);

export const handler: APIGatewayProxyWebsocketHandlerV2 =
  async (event: APIGatewayProxyWebsocketEventV2): Promise<APIGatewayProxyResultV2> => {
    console.log('Event:', JSON.stringify(event));

    const deleteCommand = new DeleteCommand({
      TableName: process.env.CONN_TABLE_NAME,
      Key: {
        connectionId: event.requestContext.connectionId,
      },
    });
    const resp = await dynamoDBDocClient.send(deleteCommand);
    console.log(`deleteCommand resp => ${JSON.stringify(resp)}`);

    return { statusCode: 200 };
  };
```

<br/>

### REST API Lambda Handler
As mentioned before, our REST API Lambda will scan the DynamoDB table to get all the connections for sending push notifications. We will be using [ApiGatewayManagementApiClient](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-apigatewaymanagementapi/classes/apigatewaymanagementapiclient.html)
from AWS SDK.

```typescript
import { APIGatewayProxyEventV2, APIGatewayProxyResultV2 } from 'aws-lambda';
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient, ScanCommand } from '@aws-sdk/lib-dynamodb';
import { ApiGatewayManagementApiClient, PostToConnectionCommand } from '@aws-sdk/client-apigatewaymanagementapi';
import { TextEncoder } from 'util';

const dynamoDBClient = new DynamoDBClient({
  region: process.env.AWS_REGION,
});
const dynamoDBDocClient = DynamoDBDocumentClient.from(dynamoDBClient);

const apiGwManApiClient = new ApiGatewayManagementApiClient({
  region: process.env.AWS_REGION,
  endpoint: process.env.WS_API_ENDPOINT,
});

export const handler = async function (event: APIGatewayProxyEventV2): Promise<APIGatewayProxyResultV2> {
  console.log(`event => ${JSON.stringify(event)}`);

  const scanCommand = new ScanCommand({
    TableName: process.env.CONN_TABLE_NAME,
  });
  const scanCommandResp = await dynamoDBDocClient.send(scanCommand);
  console.log(`scanCommand resp => ${JSON.stringify(scanCommandResp)}`);

  const textEncoder = new TextEncoder();
  const connectionItems = scanCommandResp.Items || [];
  for (let ind = 0; ind < connectionItems.length; ind++) {
    const postToConnectionCommandResp = await apiGwManApiClient.send(new PostToConnectionCommand({
      ConnectionId: connectionItems[ind].connectionId,
      Data: textEncoder.encode('A new review published for book with id 123'),
    }));
    console.log(`postToConnectionCommand resp => ${JSON.stringify(postToConnectionCommandResp)}`);
  }
  return {
    statusCode: 200,
    body: JSON.stringify({
      id: 123,
      review: 'awesome book',
    }),
  };
};
```

- `endpoint`: It will be in the form of `https://{api-id}.execute-api.{region}.amazonaws.com/{stage}` which we are passing from CDK side into Lambda as environment variable.
- [`PostToConnectionCommand`](https://docs.aws.amazon.com/AWSJavaScriptSDK/v3/latest/clients/client-apigatewaymanagementapi/classes/posttoconnectioncommand.html): This is the command object we need to use for sending messages to connections. Hence the fact that,
`Data` field expects an encoded string instead of raw string value, so we need to use `TextEncoder` to encode the data. If we are sending a JSON object then we need to first serialize it into a string and then encode it.


## CDK Code
We will be provisioning a `DynamoDB` table, three `Lambda` functions, one `WebSocketApi` and one `RestApi`.

```typescript
import * as cdk from 'aws-cdk-lib';
import * as apigw from 'aws-cdk-lib/aws-apigateway';
import * as apigw2 from '@aws-cdk/aws-apigatewayv2-alpha';
import * as apigw2Integrations from '@aws-cdk/aws-apigatewayv2-integrations-alpha';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import * as lambdaNodejs from 'aws-cdk-lib/aws-lambda-nodejs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';

export class AppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const connectionTable = new dynamodb.Table(this, 'main-dynamo-table', {
      partitionKey: {
        name: 'connectionId',
        type: dynamodb.AttributeType.STRING,
      },
      removalPolicy: cdk.RemovalPolicy.DESTROY,
    });

    const wsConnectLambda = new lambdaNodejs.NodejsFunction(this, 'ws-connect-lambda', {
      entry: './src/ws-connect-lambda.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_18_X,
      environment: {
        CONN_TABLE_NAME: connectionTable.tableName,
        NODE_OPTIONS: '--enable-source-maps',
      },
    });
    connectionTable.grantWriteData(wsConnectLambda);

    const wsDisconnectLambda = new lambdaNodejs.NodejsFunction(this, 'ws-disconnect-lambda', {
      entry: './src/ws-disconnect-lambda.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_18_X,
      environment: {
        CONN_TABLE_NAME: connectionTable.tableName,
        NODE_OPTIONS: '--enable-source-maps',
      },
    });
    connectionTable.grantWriteData(wsDisconnectLambda);

    const webSocketApi = new apigw2.WebSocketApi(this, 'websocket-api', {
      connectRouteOptions: {
        integration: new apigw2Integrations.WebSocketLambdaIntegration('ws-connect-integration', wsConnectLambda),
      },
      disconnectRouteOptions: {
        integration: new apigw2Integrations.WebSocketLambdaIntegration('ws-disconnect-integration', wsDisconnectLambda),
      },
    });

    const webSocketStage = new apigw2.WebSocketStage(this, 'websocket-stage', {
      webSocketApi: webSocketApi,
      stageName: 'prod',
      autoDeploy: true,
    });

    const restApi = new apigw.RestApi(this, 'rest-api', {
      deployOptions: {
        stageName: 'prod',
      },
      deploy: true,
      defaultCorsPreflightOptions: {
        allowMethods: ['POST', 'OPTIONS'],
        allowOrigins: apigw.Cors.ALL_ORIGINS,
      },
    });

    const apiLambda = new lambdaNodejs.NodejsFunction(this, 'api-lambda', {
      entry: './src/api-lambda.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_18_X,
      environment: {
        WS_API_ENDPOINT: `https://${webSocketApi.apiId}.execute-api.${props?.env?.region}.amazonaws.com/${webSocketStage.stageName}`,
        CONN_TABLE_NAME: connectionTable.tableName,
        NODE_OPTIONS: '--enable-source-maps',
      },
    });
    connectionTable.grantReadData(apiLambda);
    webSocketApi.grantManageConnections(apiLambda);

    restApi.root.addMethod('POST', new apigw.LambdaIntegration(apiLambda));
  }
}
```

<br/>

## Testing
Our testing phase will consist of two steps. We will connect to our API Gateway Websocket Server from command line. And send a POST request from another terminal to verify a post notification is being sent to the online client.

### Connect to Websocket Server
We will be using [`wscat`](https://www.npmjs.com/package/wscat) module for this. You can either install it as a global library from npm or homebrew.

```shell
> wscat -c wss://8po1pcbuke.execute-api.us-east-1.amazonaws.com/prod
Connected (press CTRL+C to quit)
```

After a successful connection is initiated we can also verify that the related `connectionId` exists in our DynamoDB table.

```shell
> aws dynamodb scan --table-name AppStack-maindynamotableAB55E21F-8QAZTQJQFQ6C --region us-east-1

{
  "Items": [{
    "connectionId": {
      "S": "CzRC9d73IAMCIZA="
    }
  }],
  "Count": 1,
  "ScannedCount": 1,
  "ConsumedCapacity": null
}
```

### Publish a Notification
We will send a POST request from a new terminal and our API Lambda will get all the available connections, send a push notification to each available connection and return back a JSON response.

```shell
> curl -i -X POST 'https://y69qybuse7.execute-api.us-east-1.amazonaws.com/prod/'

{"id":123,"review":"awesome book"}
```

After sending the request, a new message should pop-up from the terminal window with a message `A new review published for book with id 123`.

### Disconnect from Websocket Server
We will close the terminal window to disconnect from Websocket Server and verify that there are no any connection items in the DynamoDB table.

```shell
> aws dynamodb scan --table-name AppStack-maindynamotableAB55E21F-8QAZTQJQFQ6C --region us-east-1

{
  "Items": [],
  "Count": 0,
  "ScannedCount": 0,
  "ConsumedCapacity": null
}
```

<br/>

## Synth Stack(s)
```shell
> cdk ls

AppStack
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

## Final Note
We implemented a very simple application for sending push notifications. However, there is one important point that we didn't handle which is security. As you can see, all of our APIs (Websocket and REST) are public and not authorizing the inbound requests. There are a
couple of ways to add authorization into Websocket requests;
- Using a `REQUEST` type Lambda Custom Authorizer for `$connect` route (remember my previous [post]({{ site.url }}/api-gateway-custom-lambda-authorizer-aws-cdk/))
- Handle the authorization logic inside the `$connect` route handler by passing a token with `Sec-WebSocket-Protocol` header. This was the way I used in one of our prod application, and it worked as expected without any issues. For more information you can follow the AWS [guide](https://docs.aws.amazon.com/apigateway/latest/developerguide/websocket-connect-route-subprotocol.html).

You can also find my related github repository [here](https://github.com/flexelem/aws-cdk-examples/tree/main/api-gateway-websocket-api-example).

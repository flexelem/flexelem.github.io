---
title:  API Gateway Add Base Path Mapping into Existing Custom Domain with AWS CDK
author: buraktas
layout: post
permalink: /api-gateway-add-base-path-mapping-into-existing-custom-domain-aws-cdk/
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
comments: true
---

In this tutorial we will learn how to add base path mappings into an existing custom domain on API Gateway. Managing shared infrastructure in a separate stack is one of the most common ways in software world. We generally create this
stack for provisioning resources like databases, VPCs, domains etc. which are most likely to be referenced from another stacks belonging to different teams. And one day I was searching a way to apply this same flow for API Gateway to
make it extensible for adding multiple base path mappings into same custom domain which I struggled with a few days. However, I am not sure if this is a good practice. Anyway, to illustrate this approach we will have two different stacks;
- **InfrastructureStack**: To create a new `Certificate` for our domain, `DomainName` for generating an alias target for API Gateway and an `ARecord` to make Route53 map inbound requests to the generated API Gateway domain.
- **AppStack**: To import the `DomainName` which will be provided when creating the `BasePathMapping`.

## Infrastructure Stack
We will proceed by provisioning resources step by step in this stack to clearly understand what happens behind the scenes.

Firstly, we will import the `HostedZone` by our domain name (the value itself is imported from cdk context). Keep in mind that, I am using a domain registered from Route53 not a domain that created from another registrar. And I am getting

```typescript
// get hard-coded domainName from cdk context
const stackProps = this.node.tryGetContext('stackProps');

// Import existing hosted zone (registered domain)
const hostedZone = route53.HostedZone.fromLookup(this, 'hosted-zone', { domainName: stackProps.domainName });
```

Now we have our `hostedZone` construct in our hand and the next step will be creating a `Certificate` which will be auto-approved since we are using a domain created and managed from Route53 itself. Otherwise,
there are a few validation types that we need to take manual actions and cdk deploy won't proceed until it is validated.

```typescript
// create a new certificate which will be auto approved since we own the domain
const certificate = new acm.Certificate(this, 'main-domain-certificate', {
  domainName: stackProps.domainName,
  validation: acm.CertificateValidation.fromDns(hostedZone),
});
```

We will assign this created certificate to our existing custom domain via `DomainName` construct.

```typescript
// create an API Gateway custom domain which will be added into ARecord
const domainName = new apigw.DomainName(this, 'domain-name', {
  domainName: stackProps.domainName,
  certificate: certificate,
  endpointType: apigw.EndpointType.REGIONAL,
  securityPolicy: apigw.SecurityPolicy.TLS_1_2,
});
```

Now if we deploy this infrastructure stack at this point then we will see 2 resources created with behaviours;
- Certificate is attached to our domain.
- A custom API Gateway domain is created without any mapping into it since there is no any associated alias record existing from our HostedZone's DNS record set yet.


Verify that a new Certificate created and assigned to our domain.
```shell
> aws acm list-certificates --region us-east-1

{
  "CertificateSummaryList": [{
    "CertificateArn": "arn:aws:acm:us-east-1:************:certificate/6d567b47-32b5-4f59-925b-3602e466e290",
    "DomainName": "awesome-cdk-examples.com",
    "SubjectAlternativeNameSummaries": [
      "awesome-cdk-examples.com"
    ],
    "HasAdditionalSubjectAlternativeNames": false,
    "Status": "ISSUED",
    "Type": "AMAZON_ISSUED",
    "KeyAlgorithm": "RSA-2048",
    "KeyUsages": [
      "DIGITAL_SIGNATURE",
      "KEY_ENCIPHERMENT"
    ],
    "ExtendedKeyUsages": [
      "TLS_WEB_SERVER_AUTHENTICATION",
      "TLS_WEB_CLIENT_AUTHENTICATION"
    ],
    "InUse": true,
    "RenewalEligibility": "ELIGIBLE",
    "NotBefore": "2023-03-21T03:00:00+03:00",
    "NotAfter": "2024-04-19T02:59:59+03:00",
    "CreatedAt": "2023-03-21T19:33:48.129000+03:00",
    "IssuedAt": "2023-03-21T19:34:18.147000+03:00"
  }]
}
```

<br/>

An API Gateway custom domain is created with neither having any base path mappings and mapped from any alias (ARecord) records.
```shell
> aws apigatewayv2 get-domain-names --region us-east-1

{
  "Items": [{
    "ApiMappingSelectionExpression": "$request.basepath",
    "DomainName": "awesome-cdk-examples.com",
    "DomainNameConfigurations": [{
      "ApiGatewayDomainName": "d-ih6y4cxj26.execute-api.us-east-1.amazonaws.com",
      "CertificateArn": "arn:aws:acm:us-east-1:************:certificate/6d567b47-32b5-4f59-925b-3602e466e290",
      "DomainNameStatus": "AVAILABLE",
      "EndpointType": "REGIONAL",
      "HostedZoneId": "Z1UJRXOUMOOFQ8", // This is the hostedZoneId generated for 'ApiGatewayDomainName' NOT from our main domain
      "SecurityPolicy": "TLS_1_2"
    }]
  }]
```

<br/>

```shell
> aws route53 list-resource-record-sets --hosted-zone-id Z04491901AJOYW36491VV --query "ResourceRecordSets[?Type == 'A']

[]
```

<br/>

Alright, we are now moving forward with adding a new alias (ARecord) record to make Route53 point to the newly created API Gateway custom domain which is `d-ih6y4cxj26.execute-api.us-east-1.amazonaws.com`.

```typescript
// add a new ARecord into our hosted zone DNS records to point API Gateway domain
const aRecord = new route53.ARecord(this, 'domain-name-arecord', {
  zone: hostedZone,
  recordName: stackProps.domainName,
  target: route53.RecordTarget.fromAlias(new targets.ApiGatewayDomain(domainName)),
});

new cdk.CfnOutput(this, 'domain-name-alias-domain-name', {
  value: domainName.domainNameAliasDomainName,
  description: 'alias domain name of the domain name',
  exportName: 'domainNameAliasDomainName',
});
```

Note that, `target` from `ARecord` is an actual construct which expects `DomainName` construct to be provided. Our Route53 is now configured to map inbound requests to API Gateway domain with created new `A` alias record.

```shell
> aws route53 list-resource-record-sets --hosted-zone-id Z04491901AJOYW36491VV --query "ResourceRecordSets[?Type == 'A']

[{
  "Name": "awesome-cdk-examples.com.",
  "Type": "A",
  "AliasTarget": {
    "HostedZoneId": "Z1UJRXOUMOOFQ8",
    "DNSName": "d-ih6y4cxj26.execute-api.us-east-1.amazonaws.com.",
    "EvaluateTargetHealth": false
  }
}]
```

As the final step, we have to export `domainName.domainNameAliasDomainName` value which will be assigned to `domainNameAliasTarget` property when importing `DomainName` from our **AppStack**. This was the place creating confusion to me because
there wasn't any place that AWS clearly mentions this in their docs. Instead, from their CDK [DomainName](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.DomainName.html#domainnamealiasdomainname) documentation there is a description stating that
 `domainName.domainNameAliasDomainName` is a Route53 alias target in order to connect a record set to this domain through an alias. This is exactly the same description used for [`domainNameAliasTarget`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.DomainNameAttributes.html) from
[`DomainNameAttributes`](https://docs.aws.amazon.com/cdk/api/v2/docs/aws-cdk-lib.aws_apigateway.DomainNameAttributes.html).

<br/>

## App Stack
We have everything ready for provisioning our api by adding a new base path mapping. We just need to import our API Gateway domain which is a shared resource. There won't be any incremental steps we will follow to deploy instead we will just deploy our stack as a whole.

```typescript
import * as cdk from 'aws-cdk-lib';
import * as route53 from 'aws-cdk-lib/aws-route53';
import * as apigw from 'aws-cdk-lib/aws-apigateway';
import * as lambdaNodejs from 'aws-cdk-lib/aws-lambda-nodejs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import { Construct } from 'constructs';

export class AppStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    const stackProps = this.node.tryGetContext('stackProps');

    // Import existing hosted zone (registered domain)
    const hostedZone = route53.HostedZone.fromLookup(this, 'hosted-zone', { domainName: stackProps.domainName });

    // Import domainName we created in 'InfrastuctureStack'
    const domainName = apigw.DomainName.fromDomainNameAttributes(this, 'domain-name', {
      domainName: stackProps.domainName,
      domainNameAliasHostedZoneId: hostedZone.hostedZoneId,
      domainNameAliasTarget: cdk.Fn.importValue('domainNameAliasDomainName'),
    });

    const orderRestApi = new apigw.RestApi(this, 'rest-api-order', {
      deployOptions: {
        stageName: 'prod',
      },
      deploy: true,
      defaultCorsPreflightOptions: {
        allowMethods: ['GET', 'OPTIONS'],
        allowOrigins: apigw.Cors.ALL_ORIGINS,
      },
    });

    // add '/order' base path mapping
    new apigw.BasePathMapping(this, 'base-path-mapping-order', {
      basePath: 'order',
      domainName: domainName,
      restApi: orderRestApi,
    });

    const orderApiLambda = new lambdaNodejs.NodejsFunction(this, 'awesome-api-lambda', {
      entry: './src/order-api-lambda.ts',
      handler: 'handler',
      runtime: lambda.Runtime.NODEJS_18_X,
      environment: {
        NODE_OPTIONS: '--enable-source-maps',
      },
    });

    // /order
    orderRestApi.root.addMethod('GET', new apigw.LambdaIntegration(orderApiLambda));
  }
}
```

To verify we successfully created a base path mapping with path `/order`;
```shell
> aws apigatewayv2 get-api-mappings --domain-name awesome-cdk-examples.com --region us-east-1

{
  "Items": [{
    "ApiId": "5xdi73zy8k",
    "ApiMappingId": "rjem9p",
    "ApiMappingKey": "order",
    "Stage": "prod"
  }]
}
```

Finally, we should get a success response from our api Lambda fronted by our custom domain.

```shell
> curl -i -X GET 'https://awesome-cdk-examples.com/order'

HTTP/2 200
date: Wed, 22 Mar 2023 12:40:17 GMT
content-type: application/json
content-length: 28
x-amzn-requestid: 9e3c1152-8dc6-4f12-9e25-d33614bcb848
x-amz-apigw-id: CLuPqGBHoAMFd4g=
x-amzn-trace-id: Root=1-641af730-7ce2f0732882625932990183;Sampled=0

{"id":123,"category":"book"}
```

Same approach can be applied to subdomains or even `RestApi` constructs to be imported from different stacks.

<br/>

## Synth Stack(s)
```shell
> cdk synth

Supply a stack id (InfrastuctureStack, AppStack) to display its template
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

Here you can find the related github [repository](https://github.com/flexelem/aws-cdk-examples/tree/main/add-base-path-mapping-into-existing-custom-domain-example).

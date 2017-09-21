---
layout: post
title: Create A Simple API With CloudFormation DynamoDB And ApiGateway
description: A post introducing how to quickly set up a simple api on ApiGateway by CloudFormation
tags: AWS CloudFormation Apigateway api DynamoDB 
---

Recently, I spent most of my time learning and using AWS services, and in this post, I am going to show how to create a very simple API function with CloudFormation, DynamoDB and API Gateway.

AWS CloudFormation is an AWS service that allows you to set up AWS resources in AWS. You just need to create a template which describes all AWS resources you need, and AWS CloudFormation will take care of the rest of things like provisioning and configuring resources. In general, CloudFormation uses files in the format of JSON or YAML to create templates (all codes in the posts are in YAML format).

API Gateway enables you to create, publish, maintain and secure APIs in AWS. It can be considered as a backplane in the cloud to connect AWS services and other websites.

DynamoDB is a fully managed NoSQL database service with which you can create database tables that can store and retrieve data and serve request traffic.

The core idea in this project is to create a CloudFormation template that allows API Gateway to read data from AWS DynamoDB, and push data out once there is a call. First let's take a look at the elements in CloudFormation. The mandatory and most important one is the resource, and optionally, we can define parameters that enable us to specify values for use in properties of resources.

For instances, we can define such parameters:

```yaml
Parameters:
  ReadCapacityUnits:
    Description: Provisioned read throughput
    Type: Number
    Default: '500'
    MinValue: '5'
    MaxValue: '10000'
    ConstraintDescription: must be between 5 and 10000
  WriteCapacityUnits:
    Description: Provisioned write throughput
    Type: Number
    Default: '500'
    MinValue: '5'
    MaxValue: '10000'
    ConstraintDescription: must be between 5 and 10000
  TableName:
    Type: String
    Default: customers_table
  ApiName:
    Type: String
    Default: customers_api
    Description: ‘creation of api’
```

After defining parameters, we can start defining all resources we need to use. Here is the flow of the project, for the DynamoDB, we just need "AWS::DynamoDB::Table". And for API Gateway, the condition is more complex. First of all, we define an API container to group all other stuff, the type of resource for it is "AWS::ApiGateway::RestApi". If we want API to expose under GET/branch (branch is just a name, you can assign any valid name for it), we need to describe a HTTP resource for this, like "AWS::ApiGateway::Resource"; If we want API to expose just under GET/, then we don't need to add this resource. Then, we will define the HTTP method for API where we are going to use “AWS::ApiGateway::Method”. Remember that each time when you make the basic settings for API, you have to deploy it, otherwise, the settings will not be updated. Thus we use “AWS::ApiGateway::Deployment” to achieve this function. Now, we have made it clear of what basic resources we need to set up this simple API function in CloudFormation, so let’s see the YAML codes for them.

The first part is for definition of DynamoDB table which is relatively easy since for now, only one resource is needed.
```yaml
Resources:

  CustomerInfo:
    Type: 'AWS::DynamoDB::Table'
    Properties:
      TableName: !Ref 'TableName'
      KeySchema:
        - AttributeName: MovieName
           KeyType: HASH
        - AttributeName: Year
           KeyType: RANGE
      AttributeDefinitions:
        - AttributeName: CustomerID
           AttributeType: S
        - AttributeName: Year
           AttributeType: N
      ProvisionedThroughput:
        ReadCapacityUnits: !Ref 'ReadCapacityUnits'
        WriteCapacityUnits: !Ref 'WriteCapacityUnits'
```

The second part will be for Api Gateway, there are four resources used in this example, and all of resources should be placed in the same level with the one for DynamoDB, and here are how I define these resources:

```yaml
  CustomerApi:
    Type: 'AWS::ApiGateway::RestApi'
    Properties: 
      Ref: ApiName
    DependsOn:
      - CustomerInfo

  CustomerResource:
    Type: "AWS::ApiGateway::Resource"
    Properties:
      RestApiId:
        Ref: CustomerApi
      ParentId:
        Fn::GetAtt:
          - CustomerApi
          - RootResourceId
      PathPart: "branch”

  CustomerGet:
    DependsOn: CustomerApi
    Type: 'AWS::ApiGateway::Method'
    Properties:
      RestApiId: 
        Ref: CustomerApi
      ResourceId: 
        Ref: CustomerResource
      HttpMethod: GET
      AuthorizationType: NONE
      Integration:
        IntegrationHttpMethod: POST
        Credentials: 'arn:aws:iam::******:role/******’
        Type: AWS
        Uri: !Sub "arn:aws:apigateway:${AWS::Region}:dynamodb:action/Scan"
        RequestTemplates:
          application/json: >-
            {******}
        IntegrationResponses:
          - ResponseTemplates:
              application/json: >-
               {*****}
            StatusCode: 200
      RequestParameters:
        method.request.querystring.**: true
        method.request.querystring.**: true
      MethodResponses:
        - ResponseModels:
            application/json: Empty
          StatusCode: 200

  CustomerDeploy:
    DependsOn: 
      - CustomerApi
      - CustomerGet
      - CustomerResource      
    Type: "AWS::ApiGateway::Deployment"
    Properties:
      RestApiId:
        Ref: CustomerApi
      Description: "Deployment for this customer api“
      StageName: ‘testing’
```
Congrats, now you have a very simple API which reads data from DynamoDB and contains basic functions an API needs to query results based on self-defined query parameters. Although these settings are not complete, this API gateway can at least now do some baisc functions as described before. Some points are still good to mention here. First, regarding the “credentials”, here I fill it in with a fixed IAM role, and it’s better to separately define it and attach the role to the credentials, I will introduce how to do it in the next post. Secondly, I leave the “application/json” template blank here, but in order to let a Rest API work, you need to have request and integrate template in a way that your API can get something from DynamoDB. Another thing when writing down the template is the query parameters have to be aligned with request parameters which are defined afterwards. When deploying the API, better depend it on all resources we use before so that this “deployment” resource will be executed only when all other resources are done. One more point needs to be added, a stage needs to be defined later, since the stage defines a path through which an API deployment is accessible. This will also be described in details in the next post. 

Tips: when defining the API action to DynamoDB, Query and Scan are most commonly used. The difference is Query only returns one result each time, and Scan can return multiple ones.







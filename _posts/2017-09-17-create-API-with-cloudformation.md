Recently, I spent most of my time learning and using AWS services, and in this post, I am going to show how to create a simple API function with CloudFormation, DynamoDB and API Gateway.

AWS CloudFormation is an AWS service that allows you to set up AWS resources in AWS. You just need to create a template which describes all AWS resources you need, and AWS CloudFormation will take care of the rest of things like provisioning and configuring resources. In general, CloudFormation uses files in the format of JSON or YAML to create templates. 

API Gateway enables you to create, publish, maintain and secure APIs in AWS. It can be considered as a backplane in the cloud to connect AWS services and other websites.

DynamoDB is a fully managed NoSQL database service with which you can create database tables that can store and retrieve data and serve request traffic.

The core idea in this project is to create a CloudFormation template that allows API Gateway to read data from AWS DynamoDB with IAM role and policy. First let's take a look at the elements in CloudFormation. The mandatory and most important one is the resource, and optionally, we can define parameters that enable us to specify values for use in properties of resources.

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
    Default: tmp_pwt
  ApiName:
    Type: String
    Default: tmp_pwt_api_2
    Description: 'tmp for api'
```


After defining parameters, we can start defining all resources we need to use. Here is the flow of the project, for the DynamoDB, we just need "AWS::DynamoDB::Table". And for API Gateway, the condition is more complex. First of all, we define an API container to group all other stuff, the type of resource for it is "AWS::ApiGateway::RestApi". If we want API to expose under GET/branch, we need to describe a HTTP resource for this,like "AWS::ApiGateway::Resource"; If we want API to expose under GET/, then we don't need to add this resource. 
 



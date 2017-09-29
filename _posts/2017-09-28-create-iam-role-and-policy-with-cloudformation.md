In the last blog, I introduced how to build up a simple API with CloudFormation, ApiGateway and DynamoDB, this time, I will bring you some more complex settings of API, and show you how to add IAM role and policy in the meanwhile for your AWS project in your CloudFormation template.

I will first bring in the concept of IAM. AWS IAM which stands for Identity and Access Management, is an AWS web service to securely control access to AWS resources for users. By providing standalone IAM policies, AWS can then grant necessary permission for common use cases so that there is no need for you to investigate what permissions are needed. You can attach your policies to IAM users or groups that require those permissions.

Two statements need to address, one is the ‘Action’ in which you grant the permissions, if you fill it in with ‘’dynamodb: * ’’, it means you grant full DynamoDB actions to DynamoDB resources. But if you crate a policy that is more restrictive than the minimum required permissions for a user to work with DynamoDB, some functions may not work. Another one is the ‘Resource’ field, you specify the table to which the permissions apply. Generally, you fill in with the Amazon Resource Name (ARN) which identifies a table in a specific region. The structure of the ARN is “arn:aws:dynamodb:{region}:{account_number}:table/{table_name}”, this will allow policies only on the table that exists with a specified name, if you replace table name with a wildcard character, it will mean that you grant the permissions to all tables in this account. A useful tip when specifying tables as resource is, you can use a wildcard character to grant permissions on all tables where the name is prefixed with the name of IAM user that is making the request, it looks like this: “Resource” : “arn:aws:dynamodb:eu-west-1: {account_number}:table/{aws_username}_*”

Now let’s see a simple example for the demonstration and to show you how this works with CloudFormation.
```yaml
  CustomerPolicy:
    Type: "AWS::IAM::Policy"
    Properties:
      PolicyName: “customer_policy”
      PolicyDocument:
        Version: "2012-10-17"
        Statement:
          -
            Effect: "Allow"
            Action:
              - "dynamodb:BatchGetItem"
              - "dynamodb:DescribeTable"
              - "dynamodb:GetItem"
              - "dynamodb:ListTables"
              - "dynamodb:Query"
              - "dynamodb:Scan"
            Resource: ‘arn:aws:dynamodb:eu-west-1:*****:table/****’
```
Note that, the version of the policy is recommended to be specified to 2012-10-17, and the default version of this (2008-10-17) doesn’t support policy variables. Furthermore, if you would like to attach the policy to an IAM role, you need to specify the ‘Roles’, for instance, assume that we have created a role call CustomerRole, then here is how we attach CustomerPolicy to this role:

```yaml
CustomerPolicy:
  Type: “AWS::IAM::Policy”
  Properties:
    PolicyName: ***
    PolicyDocument:
      Version: “2012-10-17”
      Statement:
       -
         Effect: **
         Action: ****
         Resource: ****
  Roles:
    -
      Ref: CustomerRole
```
One import point I need to mention is that the Roles here only accepts a list of string, so that’s why we need to add a bar(“-”) there which has the same function as a list! Another tip I would like to mention is “DependsOn”, since practically, if you want to attach a policy to a role, the role has to be created first, otherwise, you might get an error. In order to deal with it and make sure the code is executed sequentially, we use “DependsOn” to let the resource be executed dependently, that is, in this case, if you want this Policy resource to be executed after the Role resource is generated, you can add a “DependsOn” to this CustomerPolicy, it looks like this:
```yaml
CustomerPolicy:
  DependsOn:
    -  CustomerRole
  Type: **
  Property: **
```
Let’s now step to how we can create an IAM role with CloudFormation, it’s pretty simple and similar to creating policy:
```yaml
  CustomerRole:
    Type: 'AWS::IAM::Role'
    Properties:
      RoleName: customer_role
      AssumeRolePolicyDocument:
        Version: '2012-10-17'
        Statement:
          - 
            Effect: Allow
            Principal:
              Service:
                - ‘s3.amazonaws.com'
            Action: 'sts:AssumeRole'
```
In the service section, we can fill it in with the service where we want to apply this role, this can be multiple.

Now, an IAM role attached by an IAM policy has been created and specified.

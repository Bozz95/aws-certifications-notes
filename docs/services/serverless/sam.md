# AWS Serverless Application Model (SAM) <!-- omit in toc -->

- [Description](#description)
- [Templates](#templates)
- [Deploy](#deploy)


## Description

It consists in a framework used to create and manage your serverless applications.

You use YAML to configure SAM and then it all gets converted into Cludformations templates which will follow the AWS best practices by design.

Since it ends up using Cloudformation it supports all its standard functionalities such as Outpus, Mappings, Parameters...

You can depoy Lambda function using SAM, which will then rely on CodeDeploy.

SAM can let you test your stack locally emulating Lambda, DynamoDB and API Gateway.

## Templates

SAM rely on "recipes" or templates, which is indicated in the `transform` header.

Instead of using the Cloudformation directive you have to use the SAM one to declare Functions and so on.

- `AWS::Serverless::Function` -> Lambda
- `AWS::Serverless::Api` -> Api Gateway
- `AWS::Serverless::SimpleTable` -> DynamoDB

## Deploy

Using the command `sam deploy` you can package (used to be a separate command) and deploy your serveless infra.

You can rapidly synch you lambda code with the local changes using `sam synch --watch`.

> Note: `synch` bypasses the transformation into cloudformation code, it just load the lambda code into the function to make it faster.

# Service Catalog <!-- omit in toc -->

- [Intro](#intro)
- [Terminology](#terminology)
- [Stack Set Constraints](#stack-set-constraints)
- [Launch Constraints](#launch-constraints)
- [CICD](#cicd)

## Intro

Is more like a methodolgy, when you want to deploy a lot of resources or stacks into your AWS Account you want those to be compliant for security and well architechted.
<br>To have a list of Services created according to those standard AWS offers the `Service Catalog ` service.

In syntax service Catalog is a aggregtion of CF templates.

## Terminology

- Sevice Catalog - configuration of IT Services all via CF
- `Products` are simply the CF Templates, so having that catalog helps user to more consistent and standarzide
- You then DO NO give users permissions to run CF templates directly, but instead you give permissions to use the Service Catalog, which underneath uses Cloudformation.

## Stack Set Constraints

You can specify contraints on stackset to manage or restrict the avilability of your prducts based on:

- **Accounts** where you want to make your products available
- **Region** where your stack can be deployed
- **IAM Roles** that can manage that target account

## Launch Constraints

With this type of contraints you can specify the IAM Role assegned to the products and the minimal required permissions to launch it.

## CICD

Simple CICD will listen to events on the code repo, i.e. Codecommit, a Lambda will be launched on every commit on said repo and using a mapping **Product/Account-Region** where the pipelin will know where to deploy the new version.

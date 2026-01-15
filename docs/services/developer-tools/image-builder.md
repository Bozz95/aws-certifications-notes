# EC2 Image Builder

## Intro

Automated system where you can build and test your EC2 AMI.

The service itself is free but you just pay the underline resoruce upon which it runs, which are basically EC2 instances.

Can publish AMI to multiple regions and multiple Accounts.

It's typical workflow is like this:

1. Image Builder creates the instance to run the job
2. The builder builds the AMI image
3. It generate the new AMI
4. it loads a new EC2 with that AMI to test it
5. After the tests are positive it publishes it

## RAM - Resource Access Manager

Is another service used with `Image Builder` to share Images, Recipes and Components across AWS accounts.

## Tracking Latest AMI

Using SSM parameters to share into the account and across them the laster build version.

Same mechanis used in the terraform EKS module.

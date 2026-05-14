# CDK <!-- omit in toc -->

- [Intro](#intro)
- [SAM vs CDK](#sam-vs-cdk)

## Intro

Framework created on top of Cloudformation.

Purpose: make developers use their programming language without needing them to learn CF.

On top of it using a programming language you have the advantage to have all type chacking and compiling check beforehand, so less malformed apply to cloudformation.

## SAM vs CDK

SAM is serverless only, less in depth, quick start for lambda and other lambda dependent stack.

CDK allow the use of every (supported by the library) AWS service, and you can use whatever language you are more familiar with, that is supported.

You can combine SAM and CDK, since CDK creates a CF template you can check serverless infra using the `cdk synth` command and then make SAM check it.

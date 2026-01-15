# Cloudformation <!-- omit in toc -->

- [Intro](#intro)
- [Templates](#templates)
  - [Temmplate Building Blocks](#temmplate-building-blocks)

## Intro

AWS goto service for IaC management of resources, it uses yaml files to configure resources.

Each `Stack`, which is a group of resource in a Cloudformation template, is tagged by an identifier which can be used to easily track cost by Stack.

Also using Cloudformation stack AWs has services to estimate its cost.

Other benefits:

- Productivity and reproducibility increase
- Auto generation of diagrams schemas via templates for documentation
- There are already plenty of templates you can re-use on aws documentation or repositories.

## Templates

To use templates you must save them in S3 and reference them in cloudformation, edit the template values and create the stack, then you can create resources.

It can be applied in multiple ways:

- Manually - completing the template and creating the stack via the console
- Automated way - Edit yaml files and launching them via AWS Cli or CD services

### Temmplate Building Blocks

- `AWSTemplateformatVersion` - Not required used only by AWS to indicate how to read the document
- `Description` - comments about the template purposes
- `Resources` - **THE ONY REQUIRED ONE** which contains the resources Cloudformation needs to provision
- `Parameters`- Input variables to pass to the Template
- `Mappings` - Static variables for your template
- `Outputs` - contains data about the resources created
- `Conditionals` - List of conditions upon which Cloudformations chooses which resources to create or not

There are Helpers and funcionts such the ones used in Helm charts.

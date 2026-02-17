# Cloudformation <!-- omit in toc -->

- [Intro](#intro)
- [Templates](#templates)
  - [Temmplate Building Blocks](#temmplate-building-blocks)
- [How It Works](#how-it-works)
- [Resources](#resources)
  - [Resource Blocks - FAQ](#resource-blocks---faq)
  - [Parameters](#parameters)
    - [Pseudo Parameters](#pseudo-parameters)
  - [Mappings](#mappings)
  - [Mapping vs Parameter](#mapping-vs-parameter)
  - [Outputs](#outputs)
  - [Capabilities](#capabilities)
  - [Conditions](#conditions)
  - [Intrinsic Functions](#intrinsic-functions)
    - [`Fn::Ref`](#fnref)
    - [`Fn::GetAtt`](#fngetatt)
    - [`Fn::FindInMap`](#fnfindinmap)
    - [`Fn::ImportValue`](#fnimportvalue)
  - [Deletion Policy](#deletion-policy)
  - [Stack Policies](#stack-policies)
- [RollBacks](#rollbacks)
  - [Service Role](#service-role)

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

## How It Works

New stacks are defined completely in YAML, each change or deploy will trigger some changes on stack resources.

Each resource can be created, replaced  or deleted.
**Replacing** an instance it means that the isntance needed an update but according to the resource type underline it needs to be deleted and recreted or just updated.

Parameters are like inputs to pass to the Cloudformatio template to change it without modifying the underline YAML.

## Resources

There are several tyoe of resources, more than 700 and they are constantly working on them.

Resource types are identified with this pattern:

`service-provider::service-name::data-type-name`

### Resource Blocks - FAQ

- It is possible to create a dynamic number of resources using `Macros` and `Transform`
- Not every AWS Resource is supported in AWS, very few of them are not. In case they need to be managed the answer is to use `Cloudformation Custom Resources`

### Parameters

Input for Cloudformation Templates, mandatory to make your template re-usable.

There are multiple types of parameters (String, number, list...) a few useful options are:

- `AWS Specific Parameter` - Used to validate values against existing values in AWS
- `SSM Parameters` - To manage secrets passing in template safely
- NoEcho is an option to make the user prompt the value but to keep it safe and not logging it into logs

You use parameters in the template we should use the `Fn::Ref` function, which can be used everywhere in the template. The alias `!Ref` is the same as `Fn::Ref`.

#### Pseudo Parameters

They are parameters already set up by AWS to get standard valid informations like:

- AccountID
- Region
- StackID
- StackName
- NotificaitonArn
- NoValue

### Mappings

They are fixed values withing the template, like and `Enum` in any programming language.

They are a map of string where for each key there's a fixed value assigned.

To get values from the map the `!FindInMap[MapName, TopLevelKey, SecondLevelKey]` function it is the go-to method.

### Mapping vs Parameter

Use Mapping when:

- Fixed value deduced by static variables (region, AZ,...)
- You know values in advance based on environment variables
- You want safer control on the template parametrization

Use Parameters:

- You want to grant the user the maximum level of freedom

### Outputs

Once the Cloudformation Stack is deployed it will return its output values, and they can be used to be imported in other dependent Stacks.

Best way to orchestrate work across teams that manage different stacks that need to be connected.

To leverage the import the function `!ImportValue` which needs to reference the output name, indicated by the `Export` field in the template.

Using import in one stack make it deendent on the stack from where the value is taken, so the "father" stack cannot be deleted before the "child" one. This in called `Stack link`.

### Capabilities

They are like "flags" to assign specific behavior to Cloudformtion.

- `CAPABILITY NAMED IAM` or `Capability IAM` - Used in Cloudformation stacks where IAM Resources need to be managed by the Cloudformation stack
- `CAPABILITY AUTO EXPAND` - is mandatory when used with Stack which uses Macros or Nested stack, indicates that the template may change dynamically
- `insufficient Capabilities Exceptions` -  Which is more like an alarm when capabilites need to be aknowleged before deploying it. **This is just a security measure**.

### Conditions

`Conditions` are used to control the creation of Resources or Outputs based.

Usually they are used to control which environment to crea (prod, dev...) or which regions to deploy.

A condition can reference another one or parameters or variables.
I.E.

```yaml
Conditions:
  CreateProdResources: !Equals [ !Ref EnvType, prod ]
```

This creates a condition named `CreateProdResources` which can then be used during resource creation:

```yaml
Resources:
  MountPoint:
    Type: AWS::EC2::VolumeAttachment
    Condition: CreateProdResources
```

### Intrinsic Functions

They are functions or methods used in standard languages and enabled in Cloudformation to do loop, if, get values and so on.

The most important ones are:

- `Ref` - Reference a parameter, Resouces, variables, function or something else
- `Fn::GetAtt` - Get an attribute of a resource
- `Fn::ImportValue` - Get the outout of a stack to use it in another template
- `Fn::Base64` - Function that is used to convert strings or data in Base64
- `Condition Functions` - Fn::If, Fn::Not, Fn::Equals...

#### `Fn::Ref`

Leveraged to refence values of Parameters of Resource's attributes
YAML Shorthand is `!Ref`.

#### `Fn::GetAtt`

Each resource has several attributes, all visible in the documentation, and to get their values you use this function.

While Ref returns only the UUID of the resource, i.e for EC2 ref gives the instance-id, the function `GetAtt` can give you other attributes values, like private dns, region, zone and so on.

#### `Fn::FindInMap`

Return the value of a named key in a map.

```yaml
Mappings:
  RegionMap:
  eu-south-1:
    HVM64: ami-aaaaaaaaa
    HVMG2: ami-bbbbbbbbb
  eu-south-1:
    HVM64: ami-ccccccccc
    HVMG2: ami-ddddddddd

Resources:
  MyEC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !FindInMap [RegionMap, !Ref "AWS:Region", HVM64]
      InstanceType: t4.micro
```

#### `Fn::ImportValue`

Some stack expose some values to be used in other templates.

To use those values the `ImportValue` can be used.

### Deletion Policy

You can specify what to do to provisioned resources when the stack is being deleted.

Default is `Delete`, but when needed it can be changed for safety measure or backup purposes.

With deletion policy `Retain` you can just retain the resource provisioned with Cloudformation without any major risk of being deleted during stack destruction.

Another important policy is `Snapshot`, which indicates to Cloudformation to make a Snapshot of said resource before destroying it. Beware that not every AWS Service supports this mode.

With Termination Protection you can manage which resource to NOT Delete when the stack is removed.

### Stack Policies

This policies are made to enable Cloudformation to update exisiting resources during Template deployment.

They are define via  JSON.

By default all resources in the same stack are allowed to be modified.



## RollBacks

By default Cloudformation doens't let you stay in a non-complete state so during stack creation or update it deletes every resource that was succesfully create alongside the one that made the flow fail.

If the stack was being updated Cloudformation upon fails it goes back to a previous working state, deleting every resources that wasn't on that state.

Due to this behavior the only way to debug issues in this way is to look at Cloudformatio logs or Cloudtrail Api Calls.

You can disable this behavior and keep not-completed state to look at the resources in depth at the resources that made the template fail.

That can be rollbacks failures, this is usually due to resources that cannot rolled back automatically.
So you need to fix them manually and than complete the rollback, using the `ContinueRollback`.

### Service Role

Cloudformation needs a role to operate on resource, you can use its SeriviceRole to grant user the ability to provision resources but to not directly work on them.

The user role need `cloudformation:*` and `iam:PassRole` to be able to interact with cloudformation and use its service role.

Usefull for security puporses and minimal access policy for users.

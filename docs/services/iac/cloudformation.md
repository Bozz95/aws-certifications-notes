# Cloudformation <!-- omit in toc -->

- [Intro](#intro)
- [Templates](#templates)
  - [Temmplate Building Blocks](#temmplate-building-blocks)
  - [Resource Blocks - FAQ](#resource-blocks---faq)
  - [Parameters](#parameters)
    - [Pseudo Parameters](#pseudo-parameters)
  - [Mappings](#mappings)
  - [Mapping vs Parameter](#mapping-vs-parameter)
  - [Outputs](#outputs)
  - [Conditions](#conditions)
  - [Intrinsic Functions](#intrinsic-functions)
    - [`Fn::Ref`](#fnref)
    - [`Fn::GetAtt`](#fngetatt)
    - [`Fn::FindInMap`](#fnfindinmap)
    - [`Fn::ImportValue`](#fnimportvalue)

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

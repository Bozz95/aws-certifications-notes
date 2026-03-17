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
    - [`Fn::Base64`](#fnbase64)
  - [Deletion Policy](#deletion-policy)
  - [Stack Policies](#stack-policies)
- [Custom Resources](#custom-resources)
- [Dynamic References](#dynamic-references)
- [RollBacks](#rollbacks)
  - [Service Role](#service-role)
- [Common Problems](#common-problems)
  - [Improving management for UserData script](#improving-management-for-userdata-script)
    - [`cfn-init`](#cfn-init)
    - [`cfn-signal` and Wait Conditions](#cfn-signal-and-wait-conditions)
- [Nested Stacks](#nested-stacks)
- [`Depends On`](#depends-on)
- [Stacksets](#stacksets)
  - [Permissions](#permissions)
- [Troubleshooting](#troubleshooting)
  - [Deletion failing](#deletion-failing)
  - [StackSet Troubleshooting](#stackset-troubleshooting)

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

#### `Fn::Base64`

Function used to convert long strings into base 64 and pass them to resources which will need them, like the user_script data for the EC2 instances.

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

## Custom Resources

Not every resource can be provisioned by Cloudformation, or are not yet supported.

So `Custom Resources` comes to the rescue, thei best use case scenarios are:

- Define resources not yet supported natively by CF
- Provision resources outside the AWS Domain, such as on-prem isntances or 3rd party ones (other providers)
- Create some scripts, like local exec (Terraform), to be run on creation/deletion/update of resource. I.E to empty a S3 bucket before destruction.

Syntax: `Custom::NomeOfTheCustomResource`

They are backed up the Lambda or an Sns Topic.
When defining a CR you specify a Lambda which will be called by Cloud Formation with input parameters specified in the Custom Resource definition.

Once the Lambda run successfully the Cloud Formation can continue its execution.

Lambda or Sns don't call the CF API directly, they use a pre-signed S3 bucket to store data (json response) that Cloud Formation will read.

There are libraries for send cloudformation reponses to S3.

## Dynamic References

Dynamic References are used to get values that cannot be stored in the template or change in a scope outside of it.

Normally these values are stored and can be retrieved in:

- `SSM parameters` for simple String and SecureStrings
- `SecretManager`

- SSM Syntax: `{{resolve::ssm(or ssm-secure):parameter-name:version}}` I.E: {{resolce:ssm-secure:/iam/pippo/access_key:2}}
- SecretManager: `{{resolve:secretsmanager:secret-id:secret-string:json-key:version-stage:version-id}}` I.E: {{resolve:secretsmanager:MyAccessKey:SecretString:access-access-key:value}}

Fun Fact: RDS cluster resource if created with parameter `ManageMasgerUserPassword: true` stores implicitly the admin pwd in SecretsManager, from which we can reference the arn in the outputs to get the value out.

The other way around is to create the secret ourself and point it, using the `resolve` function in the argument of the rds resource.

We can also create a attachment between RDS and the secret to establish secret rotations.

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

## Common Problems

Here a list of common problems with standard solution offered by Cloudformation to manage them according to the best practices.

### Improving management for UserData script

When there's a large UserData script or we want to make a new more readable one, or perhaps you would like to evolve the EC2 without creating a new one.

Cloudformation offers some helper Scripts:, which are python scripts:

#### `cfn-init`

It is a Block of the EC2 Metadata where the end-user can specify the configuration of the instance.

Here you can specify:

- `Packages`: which will be installed on the instance, like Python, Node, Postgres and so on.
- `Groups`: it's possible to define user groupd for the instance
- `Users`: defining users and the groups which they belong to
- `Sources`: Download files and archives to be used inside the EC2
- `Files`: Creates files inside the EC2, using inline files (like for the user_data) or pulling them from a server
- `Commands`: A series of commands
- `Services`: Launch a list of sysvinit, which are services

In practice the `csf-init`, which is a python script, command manage all these informations, interpreting all metadata and installing the specified things.

This command will communicate to Cloudformation in order to make it possible to react about the termination of the operation on the EC2.

Complex EC2 instances are gonna be more readable.

All logs of this script are gonna be savesin `/var/log/cfn-init.log`.

#### `cfn-signal` and Wait Conditions

Python script to run after the `cfn-init` to knwo if the init script was properly executed.

It used alongside a `Wait condition` which keeps the exeution of Cloudformation in "loop" untile it doesn0t receive a Signal passed. The wait condition can receive the signal from 1 or more resources, and the number of signlas needed to continue execution can be customized according to needs.

> `Wait Condition` can be customized with a timeout of x minutes.

To debug failed signalgs it is suggested to change the default behaviors of Cloudformation from `rollback on failure` to `preserve`

## Nested Stacks

Nested stacks are used when there's a need to re-use on another Cloudformation script a resource that was previously already instantated.

Nested stack are like modules in Terraform.

Difference with `Cross Stacks`:

| Cross Stacks                                            | Nested Stacks                       |
| ------------------------------------------------------- | ----------------------------------- |
| Resources in one stack have different lifecycle         | Resources have the same lifecycle   |
| Outputs are share to other stacks via `Fn::ImportValue` | Resources are directlly referenced. |

When using a NestedStack the first thing you tell Cloudformaiton is where to find the Stack template to build the nested one, and its type.

## `Depends On`

Cloudforamtion way to manage depencencies consist into the Key attributes `DependsOn` attribute.

They keyword `DependsOn` makes sure that the resource on which is attached is created after the one referenced.

## Stacksets

Stacksets in CF are used to manage stacks across multiple accounts.

It needs to define:

- **Administrator** account which will create and manage the StackSet
- **Target** accounts where the defined StackSets will execute.

When the Stackset is updated in the administrator account it will update all the target ones.

### Permissions

To manage permissions in all the accounts there are 2 ways.

- **Self-managed permissions**
  - Permissions must be created manually in each Admin and Target Account
  - Each Target account will have a trust relationship where wiht the Admin Account role
- **Service Managed Permissions**
  - Only when managing account using AWS Organizations
  - StackSets will automatically creates role on your behalf
  - All feature of AWS Organizations **MUST BE ENABLED**
  - Each account added to the OU will have the StackSet automatically deployed

## Troubleshooting

This section will talk about questions that can often be present at an exam.

### Deletion failing

When a CF stack is being delete it can fail cause the underline resources might need the resource itself to be emptied before deleting them.

For example, by default, an S3 bucket cannot be deleted if before hand the bucket is emptied.
This empty action, for this case only, must be manual (deleting file manually) or by a Custom Resource which can call a Lambda to empty the filename.

Same thing with security groups which needs to be detatched from any EC2.

### StackSet Troubleshooting

When applying a StackSet but on one or more instance of if it fails and the stack status changes to OUTDATED there can be ultiple reasons:

- Not enough Permissions in the tarted account
- The StackSet is trying to create a global resource which name needs to be UNIQUE globally, like an S3 buket.
- Missing trustrelationship across target and admin account
- Reached a limits or quota on the number of resources in that account

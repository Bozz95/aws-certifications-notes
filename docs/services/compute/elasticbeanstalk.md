# Elastic Beanstalk <!-- omit in toc -->

- [Intro](#intro)
- [Suppoerted languages](#suppoerted-languages)
- [Components](#components)
- [High Availability](#high-availability)
- [Update Kinds](#update-kinds)
- [Multiple environments](#multiple-environments)
- [Web Server vs Worker Environment](#web-server-vs-worker-environment)
- [Notification - Eventbridge](#notification---eventbridge)

## Intro

`Elastic Beanstalk` is an AWS service that takes care of creating basic web app underline infrastructure for several types of webapps architecture.

In this way developers can ignore the provisioning phase and the configuration and mostly focus on the code quality and the processes.

## Suppoerted languages

There are many supported languages. The main ones:

- Go
- Java SE
- JAva with tomcat
- .NET Core on Linux
- .NET on windows server
- Nodejs
- PHP
- Pytho

## Components

Each Beanstalk environemnt has different cmponents:

- `Application` is a Beanstalk collection of configuration to create its ideal environment
- -`Application Version` is an iteration of the source code version.
- `Environment` is a collection of AWS Resources used to run the application on beanstalk.
  Usually each enviromnet is separated based on the scenario it needs to cover.

## High Availability

Distributed through multi AZ in a VPC.

Load Balanacer in front and different EC2 on the back, anaged by an autoscaling group.

If you need a database and you create it alongside the Beanstalk environment that database will be tied to the beanstalk cluster lifecycle.

There is a choice between `Application` o `Network` Load balancer.

## Update Kinds

- `All at once`
  - fastest
  - downtime since not all instance are ready to serve data at once
  - Ideal for development phases and quick iterations
  - no additional cost
- `Rollig update`
  - few instance at a time, using "bucket size", it moves into che next batch where the previous instance is healthy.
  - No downtime but reduced capacity during update
  - multiple version running simultaneusly
  - no additional cost
  - Long deployment
- `Rolling update with additional batches`
  - No downtime ans SAME capacity due to the additional batches
  - multiple version running simultaneusly
  - additional cost
  - Long deployment
  - Spins up new instance while udpating so the old one are still availabe.
  - Good for production environment
- `Immutable`
  - No downtime, double capacity
  - Double the cost since a new equiparable rsource is created for each update
  - Spins up a new instance in thenew  ASG, does the update there and then exchange the current new version with the old one.
  - used in prod, ideal for rallback
- `Blue Green`
  - Not Native in Beanstalk
  - No downtime, double capacity
  - used in prod, ideal for rallback
  - Creates a completely new environment and switch traffic when ready using route53 weighted policies
- `Traffic Splittig`
  - the equivalent for canary testing
  - creates a completely new but equiparable environment
  - Via Load balancers can decide how much traffic send to each environment
  - One failure means the whole systemudpate will

## Multiple environments

You can configure Beansstalk to have mulitple environments for the same project (Prod, prod1, prodN, Dev, QA...), you can apply to them an update using the preferred [update kind](#update-kinds) and them swap environments to altoghether to have no downtime.

To make it possible Beanstalk offer the `clone` functionality.

## Web Server vs Worker Environment

`Web Server` mode is ideal for, as the name suggests, web servers so serving fast web pages.

But when the workload takes long to complete it is recommended to not make the web server eat those computational spikes to avoid throttling.

To make this possible is possible to decouple the application into tiers using `Worker Environments`.

For example you can configure a worker tier to poll, in an autoscaling group, an sqs queue and process the uploaded videos without impacting the web server nodes.

## Notification - Eventbridge

ElasticBeanstalk offers a lot of events to listen to:

- **Environment Operation Status** - create, update, terminate each one gives the start, success or fail event
- **Other Resources Status** - ASG, ELB, EC2 for each create/delete events
- **Manage Updates Status** - started or failed
- **Environment Health Status**

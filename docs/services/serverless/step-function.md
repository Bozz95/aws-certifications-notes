# Step Functions <!-- omit in toc -->

- [Intro](#intro)
- [Standard vs Express](#standard-vs-express)
- [Task States](#task-states)
- [States](#states)

## Intro

You can use it to model workflows state machines.

Main use:

- Data workflow, order fullfillment
- Web applications, any other asynch workflow

It uses json and you define the workflow that way.

The execution can start via:

- sdk calls
- api gateway
- Eventbridge
- ...

This service is preferred over Lambda since it can run long workflows, which in lambda are limited to 15 minutes.

## Standard vs Express

These are modes of Step Function which influence the execution speed and pricing:

- `Standard`: Long running task, up to 1 Year ideal for e-commerce automation or ETL.
- `Express`: Low cost and scaling workflows. Data processing and microservices API.

## Task States

Is a single block of the workflow, it represents an action or an invocation of an AWS Service.

Or it can run an activity, meaning it spins up an instance, run something and collect the  output.

## States

Block to influence the direction of the workflow(branch direciton):

- `Choice State` - Test a condition and send the payload to a specific branch
- `Fail or Succeed State` - Stop the execution with the chosen result fail or success
- `Pass State` - Simply pass the input to its output doing some injection or trasformation of data
- `Wait State` - Provide a delay or wait for a specific time/date
- `Map State` - Dyamically iterate steps
- `Parallel State` - **VERY IMPORTANT! Make a parallel execution branches**

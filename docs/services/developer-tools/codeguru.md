# CodeGuru

## Intro

Strumento che sfrutta meccanismo di Machile learning per effettuare review automatiche al codice, fornendo miglioramenti o consiglio di performance.

Due funzioni principali:

- `Reviewer` - Static analysis of code to find vulnerabilities or optimizations
- `Profiler` - runtime analysis about perfomance, cost optimization and so on

### Reviewer

Supports Java and Python and can integrate to Github, Bitbucket and CodeCommit.

It was trained on OpenSource projects and it is used mainly to enforce best practices and security to projects.

A cool feature is to find secrets of any kind leaked into the code or docs., and suggest remediations to ake it safer using Secret Manager.

### Profile

Identify anomalies at runtime, like excessive CPU, memory usage and detects problems.
It can improve cost usage and heap allocations.

It can run both on AWS and on-premise, it adds minimal overhead to the app.

In Python lambdas can be added as a decorator using libraries and it just need a profiing group to refer to make analysis.

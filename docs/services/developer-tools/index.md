# Developer Tools

AWS Developer Tools provide a comprehensive set of services for building, testing, and deploying applications using DevOps practices.

## Services in This Category

### [CodeBuild](codebuild.md)
Fully managed build service that compiles source code, runs tests, and produces software packages.

### [CodePipeline](codepipeline.md)
Continuous integration and continuous delivery service for fast and reliable application updates.

### [CodeCommit](codecommit.md)
Fully managed source control service that hosts secure Git repositories.

### [CodeDeploy](codedeploy.md)
Deployment service that automates application deployments to various compute services.

### [CodeStar](codestar.md)
Unified user interface for managing software development activities in one place.

## Common Use Cases

- **CI/CD Pipelines**: Automated build, test, and deployment workflows
- **Source Control**: Secure, scalable Git repositories
- **Application Deployment**: Automated deployments with rollback capabilities
- **Project Management**: Integrated development project management

## Integration Patterns

These services work together to create complete DevOps workflows:
- CodeCommit → CodeBuild → CodeDeploy
- CodePipeline orchestrating multiple services
- Integration with third-party tools (GitHub, Jenkins, etc.)

---

Tags: `#DevOps` `#CI/CD` `#AWS`
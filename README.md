# AWS ECS Fullstack App with Terraform

## Table of Contents

   * [Solution Overview](#solution-overview)
   * [General Information](#general-information)
   * [Infrastructure](#infrastructure)
      * [Infrastructure Architecture](#infrastructure-architecture)
      * [Infrastructure Considerations](#infrastructure-considerations)
      * [CI/CD Architecture](#ci/cd-architecture)
      * [Prerequisites](#prerequisites)
      * [Usage](#usage)
      * [Autoscaling Test](#autoscaling-test)
   * [Application Code](#application-code)
     * [Client App](#client-app)
     * [Client Considerations](#client-considerations)
     * [Server App](#server-app)
   * [Cleanup](#cleanup)

## Solution Overview

This repo has Terraform code to deploy a demo solution using AWS resources for a managed container infrastructure.

## General Information

The project is divided into:
- Code: Vue.js frontend and Node.js backend
- Infrastructure: Terraform code for AWS resources

## Infrastructure

The Infrastructure folder has Terraform code for AWS resources. The *Modules* folder has Terraform modules, and the *Templates* folder has configuration files. Terraform state is stored locally but can be configured for remote storage. AWS resources created include:

- Networking resources
- 2 ECR Repositories
- 1 ECS Cluster
- 2 ECS Services
- 2 Task definitions
- 4 Autoscaling Policies + Cloudwatch Alarms
- 2 Application Load Balancers
- IAM Roles and policies
- Security Groups
- 2 CodeBuild Projects
- 2 CodeDeploy Applications
- 1 CodePipeline
- 2 S3 Buckets
- 1 DynamoDB table
- 1 SNS topic

## Infrastructure Architecture

Diagram of the deployed infrastructure:

<p align="center">
  <img src="Documentation_assets/Infrastructure_architecture.png"/>
</p>

### Infrastructure Considerations

The task definition template has hardcoded memory and CPU values. Modify them using "sed" commands in CodeBuild if needed. Create an SNS topic subscriber for deployment notifications.

## CI/CD Architecture

Diagram of the CI/CD architecture:

<p align="center">
  <img src="Documentation_assets/CICD_architecture.png"/>
</p>

## Prerequisites

1. Install Terraform v0.13 or above.
2. Configure AWS credentials in ~/.aws/credentials.
3. Generate a GitHub token.

## Usage

1. Fork this repo and create a GitHub token.
2. Clone the forked repo and navigate to the Infrastructure directory:

```bash
cd Infrastructure/
```

3. Run Terraform init:

```shell
terraform init 
```

4. Run Terraform plan with required variables:

```shell
terraform plan -var aws_profile="your-profile" -var aws_region="your-region" -var environment_name="your-env" -var github_token="your-token" -var repository_name="your-repo" -var repository_owner="repo-owner"
```

Example:

```shell
terraform plan -var aws_profile="development" -var aws_region="eu-central-1" -var environment_name="dev-env" -var github_token="your-token" -var repository_name="your-repo" -var repository_owner="repo-owner"
```

5. Review and apply the plan:

```shell
terraform apply -var aws_profile="your-profile" -var aws_region="your-region" -var environment_name="your-env" -var github_token="your-token" -var repository_name="your-repo" -var repository_owner="repo-owner"
```

6. Check the AWS CodePipeline in the AWS Management Console. Add files and DynamoDB items as mentioned [here](#client-considerations). Copy the *application_url* from the Terraform output and open it in a browser.

7. Access the Swagger endpoint using the *swagger_endpoint* from the Terraform output.

## Autoscaling Test

Use Artillery for stress testing. Install it and update the **target** attribute with the ALB DNS from the Terraform output.

Frontend:

```bash
artillery run Code/client/src/tests/stresstests/stress_client.yml
```

Backend:

```bash
artillery run Code/server/src/tests/stresstests/stress_server.yml
```

Learn more about Amazon ECS Autoscaling [here](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/service-auto-scaling.html).

## Application Code

### Client App

The Client folder has Vue.js code for the frontend. It uses port 80 in production and port 3000 locally.

### Client Considerations

1. Add 3 images to the S3 bucket created by this code.
2. Add 3 DynamoDB items with the following structure:

```json
{
  "id": { "N": "1" },
  "path": { "S": "https://mybucket.s3.eu-central-1.amazonaws.com/MyImage.jpeg" },
  "title": { "S": "My title" }
}
```

### Server App

The Server folder has Node.js code for the backend. It uses port 80 in production and port 3001 locally. It has 3 endpoints:

- /status: Health check endpoint
- /api/getAllProducts: Returns all DynamoDB items
- /api/docs: Swagger API documentation

## Cleanup

Run the following command to delete all resources:

```shell
terraform destroy -var aws_profile="your-profile" -var aws_region="your-region" -var environment_name="your-env" -var github_token="your-token" -var repository_name="your-repo" -var repository_owner="repo-owner"
```
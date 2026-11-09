# Overview
This repository contains a React frontend, and an Express backend that the frontend connects to.

# Objective
Deploy the frontend and backend to somewhere publicly accessible over the internet. The AWS Free Tier should be more than sufficient to run this project, but you may use any platform and tooling you'd like for your solution.

Fork this repo as a base. You may change any code in this repository to suit the infrastructure you build in this code challenge.

# Submission
1. A github repo that has been forked from this repo with all your code.
2. Modify this README file with instructions for:
* Any tools needed to deploy your infrastructure
* All the steps needed to repeat your deployment process
* URLs to the your deployed frontend.

# Evaluation
You will be evaluated on the ease to replicate your infrastructure. This is a combination of quality of the instructions, as well as any scripts to automate the overall setup process.

# Setup your environment
Install nodejs. Binaries and installers can be found on nodejs.org.
https://nodejs.org/en/download/

For macOS or Linux, Nodejs can usually be found in your preferred package manager.
https://nodejs.org/en/download/package-manager/

Depending on the Linux distribution, the Node Package Manager `npm` may need to be installed separately.

# Running the project
The backend and the frontend will need to run on separate processes. The backend should be started first.
```
cd backend
npm ci
npm start
```
The backend should response to a GET request on `localhost:8080`.

With the backend started, the frontend can be started.
```
cd frontend
npm ci
npm start
```
The frontend can be accessed at `localhost:3000`. If the frontend successfully connects to the backend, a message saying "SUCCESS" followed by a guid should be displayed on the screen.  If the connection failed, an error message will be displayed on the screen.

# Configuration
The frontend has a configuration file at `frontend/src/config.js` that defines the URL to call the backend. This URL is used on `frontend/src/App.js#12`, where the front end will make the GET call during the initial load of the page.

The backend has a configuration file at `backend/config.js` that defines the host that the frontend will be calling from. This URL is used in the `Access-Control-Allow-Origin` CORS header, read in `backend/index.js#14`

# Optional Extras
The core requirement for this challenge is to get the provided application up and running for consumption over the public internet. That being said, there are some opportunities in this code challenge to demonstrate your skill sets that are above and beyond the core requirement.

A few examples of extras for this coding challenge:
1. Dockerizing the application
2. Scripts to set up the infrastructure
3. Providing a pipeline for the application deployment
4. Running the application in a serverless environment

This is not an exhaustive list of extra features that could be added to this code challenge. At the end of the day, this section is for you to demonstrate any skills you want to show that’s not captured in the core requirement.


# DevOps Tech Challenge 1 — AWS Deployment Documentation

## 1. Project Architecture

This project deploys a React frontend and Node.js/Express backend to AWS using Docker, Terraform, Amazon ECS Fargate, Amazon ECR, an Application Load Balancer (ALB), and Jenkins CI/CD.

The infrastructure is deployed in AWS Region `us-east-2` (Ohio).

**Architecture:**

- Terraform provisions the AWS infrastructure.
- Docker packages the frontend and backend applications into container images.
- Amazon ECR stores the Docker images.
- Amazon ECS Fargate runs the frontend and backend containers.
- The Application Load Balancer routes frontend requests to the frontend service and `/api` requests to the backend service.
- Jenkins automates building, pushing, and deploying application updates.
- Amazon CloudWatch monitors CPU utilization and supports ECS Auto Scaling.

## 2. Prerequisites and Setup

Install the following tools:

- Git
- Visual Studio Code
- Node.js and npm
- Docker
- Terraform
- AWS CLI
- Ubuntu/WSL (for running Siege on Windows)
- Siege (for load testing)

An AWS account with the required permissions is also needed.

Clone the repository:

```bash
git clone https://github.com/DBRR28/devops-tech-challenge-1.git
cd devops-tech-challenge-1
```

Configure AWS CLI credentials:

```bash
aws configure
```

Set the AWS Region to `us-east-2`.

To run the application locally, start the backend:

```bash
cd backend
npm ci
npm start
```

Then open a separate terminal and start the frontend:

```bash
cd frontend
npm ci
npm start
```

The frontend runs on port 3000, and the backend runs on port 8080.

## 3. Terraform Infrastructure Deployment

Terraform configuration files are located in the `terraform` directory.

The Terraform infrastructure includes:

- VPC, public and private subnets, and networking resources
- Security groups
- Application Load Balancer and target groups
- Amazon ECR repositories
- Amazon ECS cluster, task definitions, and services
- ECS Auto Scaling policies
- Jenkins EC2 server

To deploy the infrastructure:

```bash
cd terraform
terraform init
terraform validate
terraform plan
terraform apply
```

Review the proposed infrastructure changes and type `yes` to approve deployment.

After Terraform finishes, verify the created resources in the AWS Management Console.

**Important:** Terraform deployment requires valid AWS credentials and appropriate permissions. Review configuration variables, application URLs, and resource costs before deploying.

## 4. Jenkins CI/CD Pipeline

Jenkins runs on an Amazon EC2 instance and automates the application deployment process.

The Jenkins pipeline is defined in the `Jenkinsfile` in the repository.

**Pipeline stages:**

1. Checkout source code from GitHub.
2. Build Docker images for the frontend and backend.
3. Authenticate Docker with Amazon ECR.
4. Tag and push Docker images to ECR.
5. Trigger new Amazon ECS deployments.
6. Clean up temporary Docker resources.

**Jenkins setup:**

1. Provision the Jenkins EC2 instance using Terraform.
2. Access Jenkins through the EC2 instance's public IP on port 8080.
3. Complete Jenkins initial setup and install the necessary plugins.
4. Configure AWS permissions and Docker access.
5. Create a Jenkins Pipeline job connected to the GitHub repository.
6. Configure the pipeline to use the repository's `Jenkinsfile`.
7. Configure a GitHub webhook to trigger Jenkins automatically when code is pushed.

**Verification:**

A GitHub push successfully triggered Jenkins, and the pipeline completed with `Finished: SUCCESS`.

## 5. Application URLs

**Frontend:**

http://devops-challenge-alb-2061604971.us-east-2.elb.amazonaws.com

**Backend API:**

http://devops-challenge-alb-2061604971.us-east-2.elb.amazonaws.com/api

The ALB listens on HTTP port 80. The `/api` path is routed to the backend service.

These URLs remain available only while the corresponding AWS resources are running.

## 6. Load Testing with Siege

Siege was installed on Ubuntu 22.04 using:

```bash
sudo apt-get install siege
```

The application was tested with 250 concurrent simulated users for five minutes:

```bash
siege -c 250 -t 5M http://devops-challenge-alb-2061604971.us-east-2.elb.amazonaws.com
```

**Load test results:**

- Successful transactions: 30,359
- Failed transactions: 148
- Availability: 99.51%
- Average response time: 1.92 seconds

The application handled more than 30,000 successful transactions during the test.

## 7. ECS Auto Scaling Results

The frontend ECS service uses CPU-based target tracking.

**Configuration:**

- Target CPU utilization: 50%
- Minimum tasks: 1
- Maximum tasks: 4
- Scale-in enabled

**Verified scaling activity:**

| Time | Scaling event | Result |
|---|---|---|
| 1:28 PM | 1 → 2 tasks | Successful |
| 1:30 PM | 2 → 3 tasks | Successful |
| 1:45 PM | 3 → 2 tasks | Successful |
| 1:46 PM | 2 → 1 task | Successful |

During the Siege load test, CPU demand increased and ECS automatically scaled the frontend service from 1 to 3 tasks.

After the load test ended, CPU utilization decreased and ECS automatically reduced the service back to 1 task.

**Conclusion:** ECS Auto Scaling successfully demonstrated both scale-out and scale-in behavior.

Backend Auto Scaling was not independently verified because the load test primarily targeted the frontend through the Application Load Balancer.

## 8. Deployment Verification

The following components were successfully verified:

- Terraform infrastructure provisioning
- Docker image builds
- Amazon ECR image storage
- ECS Fargate application deployment
- ALB frontend and backend routing
- Jenkins CI/CD pipeline
- GitHub webhook automation
- Application accessibility through the ALB
- Siege load testing
- ECS automatic scale-out and scale-in

**Project Result:** Successfully deployed and automated a containerized full-stack application on AWS using Terraform, Docker, ECS Fargate, Jenkins, and CloudWatch.
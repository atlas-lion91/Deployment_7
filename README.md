# Deployment Documentation for ECS Containerized Banking Application

November 4, 2023

By: Khalil Elkharbibi


## Introduction

This document serves as a comprehensive guide for deploying a containerized banking application using Amazon Web Services (AWS) Elastic Container Service (ECS). It details the purpose, prerequisites, and a step-by-step procedure that includes best practices and common issues encountered with their resolutions. 

## Purpose

The purpose of this documentation is to outline the steps and processes involved in deploying a Banking Application in Elastic Container Service (ECS) using various DevOps tools and technologies. In modern software development, efficient and automated deployment processes are essential for several reasons:

1. **Agility and Rapid Development:** Rapid application development is a cornerstone of today's software industry. An efficient deployment process allows developers to focus on writing code while the automated pipeline takes care of building, testing, and deploying the application.

2. **Consistency and Predictability:** Manual deployments often lead to inconsistencies and unpredictable outcomes. An automated process ensures that each deployment is consistent, reducing errors and minimizing disruptions.

3. **Resource Efficiency:** Traditional infrastructure provisioning can be time-consuming and resource-intensive. With infrastructure-as-code tools like Terraform, we can provision and manage cloud resources more efficiently, enabling rapid scaling as needed.

4. **Security and Reliability:** Security is paramount in any deployment. With proper configurations, we can ensure that sensitive data is protected, and the application remains reliable.

5. **Scaling and Fault Tolerance:** Modern applications must be able to scale and recover from failures seamlessly. Using containerization and container orchestration, ECS provides the foundation for scalable, fault-tolerant deployments.

This deployment approach leverages a range of tools and services, including GitHub for version control, Amazon RDS for database management, Docker for containerization, Terraform for infrastructure provisioning, Jenkins for automation, and ECS for container orchestration. Each step in this documentation contributes to achieving these essential DevOps goals, enabling a streamlined and secure deployment process.


## Prerequisites

- AWS account with proper IAM permissions for ECS, RDS, VPC, etc.
- A GitHub account and repository for the banking application source code and Dockerfiles.
- A configured CI/CD environment with Jenkins installed and Docker and Terraform ready for use.

## System Diagram


## Steps

**VPC Configuration for ECS**
- The `vpc.tf` file in `/intTerraform` generates a VPC with:
  - 2 Availability Zones equipped with both public and private subnets for failover resilience.
  - The private subnets host the Fargate tasks containing the application in Docker containers.
  - The public subnets facilitate internet traffic routing to the private ones, mediated by an ALB.
  - A Security Group configured to permit traffic on port 8000 for private subnet communication, and port 80 for ALB internet access.
  - Two route tables directing internet-bound traffic through their respective internet and NAT gateways.
  - A NAT Gateway for outbound internet connectivity from private subnets, safeguarding against direct inbound connections.

**ECS Architecture**
- The `main.tf` script in `/intTerraform` articulates:
  - An ECS Cluster housing Fargate instances as secure hosts for the Dockerized Flask app.
  - An ECS Task Definition detailing the app container setup, including image, ports, and computing resources needed.
  - An ECS Service specifying the task quantities and load balancing configurations, ensuring service continuity and scalability.
  - A CloudWatch Log Group for logging and monitoring purposes, essential for scaling and load balancing policy formulation.

**Terraform File Adjustments for ecs-vpc**
- To adapt the `main.tf` for different deployments:
  - Assign your ECS Cluster a specific name.
  - Customize the task definition with relevant container naming and image referencing conventions from Docker Hub.
  - Configure the ECS Service and associate it with the appropriate load balancer.
  - Establish IAM roles for task execution and task definition.

**ALB Configuration via Terraform**
- The `alb.tf` manages ALB setup to secure and direct traffic:
  - Construct an ALB to interface with the Fargate-hosted containers.
  - Define listener rules to reroute traffic to the desired target group.

**GitHub Workflow and Security**
- Jenkins automation ties into GitHub through webhooks, initiating pipelines upon code changes.
- GitHub token generation for Jenkins integration involves:
  - Accessing GitHub settings to generate personal access tokens with comprehensive repository and admin hook permissions.

**Branch Strategy for Repository Management**
- Proposed branch management incorporates segregated branches for Jenkins infrastructure, container hosting setup, and application feature development.

**Docker Image Build and Test**
- Utilize the `Dockerfile` at the repository root to assemble the Flask application image.
- Test Docker image build processes locally before pushing to Docker Hub with provided Docker commands, ensuring local functionality at `localhost:8000`.

**Securing Docker Hub Integration with Jenkins**
- Configure Docker Hub access by creating and securely storing a token, then integrate this with Jenkins for Docker image pushes.

**Jenkins Setup and Pipeline Execution**
- The Jenkins ecosystem, formed within an existing VPC, uses provided Terraform scripts and consists of a trio of EC2 instances for the Jenkins setup.
- Configure the Jenkins master via shell scripts and EC2 resources, setting up SSH keys and installing necessary plugins for pipeline activities.

**Credential Management for AWS and Docker Hub**
- Add AWS and Docker Hub credentials in Jenkins for Terraform actions and Docker image deployment, using the 'Add Credentials' feature in Jenkins settings.

**Jenkins Pipeline Customization**
- Tailor the Jenkinsfile with specific Docker Hub usernames and image names, reflecting these changes at specified lines within the file.

**Incorporating Infrastructure Teardown**
- Optionally, integrate the Terraform destroy process by enabling the corresponding stage within the Jenkinsfile to tear down created resources when needed.

**Jenkins Agent Configuration**
- Setup for the Jenkins node that performs testing and builds the Docker image is scripted and made effective through resource block configurations within Terraform.

# Jenkins Deploy Agent

The Jenkins deploy agent sets up the VPC for the ECS cluster, with an application load balancer configured using the `jenkins-node2-install.sh` script located in `/Jenkins-tf`. This script is passed to Terraform within the EC2 resource block to set up the Jenkins agent service and Terraform.

## Controller and Agent Communication

To set up and configure `awsDeploy` and `awsDeploy2` agents for communication with the Jenkins controller, follow these steps:

1. Ensure the Jenkins user on the Jenkins controller has the appropriate private key at `/var/lib/jenkins/.ssh/id_rsa`.
2. On the Jenkins dashboard, navigate to `Manage Jenkins > Manage Nodes and Clouds > New Node`.
3. Name the node, select 'Permanent Agent', and click 'OK'.
4. Specify the 'Remote root directory' for the agent, such as `/home/ubuntu/agent1`.
5. In the 'Labels' field, add a label like `awsDeploy` to identify the agent.
6. For 'Launch method', choose 'Launch agent via SSH'.
7. Add the private IP address of the agent server in the 'Host' field.
8. Select the Jenkins credentials in the 'Credentials' dropdown.
9. Enter the agent server's username (e.g., `ubuntu`) in the 'Username' field.
10. In the 'Private Key' section, choose 'Enter directly' and paste the private key from the Jenkins server.
11. Save the configuration and launch the agent.

## MySQL Database in AWS RDS

To share data across application instances, use AWS RDS to create a MySQL database:

1. Go to AWS RDS on the AWS console and click 'Create database'.
2. Choose 'MySQL' and the 'Free tier' option.
3. Under 'Settings', fill in the 'DB instance identifier', 'Master username', and 'Master password'.
4. Ensure 'Public access' is set to 'Yes' and configure the security group for port 3306 ingress (egress to all traffic).
5. In 'Additional configuration', name your initial database (e.g., 'banking') and opt-out of encryption.
6. After creating the database, find it under 'Databases' and click on its name to get the endpoint from 'Connectivity & security'.

## Application Code

Update the application code to connect to the new database:

1. In `app.py`, `database.py`, and `load_data.py`, find the `DATABASE_URL` constant.
2. Replace it with the new RDS connection string format: `[mysql+pymysql://]{username}:{password}@{host}/{database-name}`.

## Deploy

Deploy the application with these git commands:

```bash
git add .
git commit -m "commit message"
git push
```

After pushing changes, the `main.tf` file's output includes the load balancer's URL for application access.

## Destroy

To remove resources:

1. Uncomment the `destroy` stage in the `Jenkinsfile`.
2. Commit and push the changes to trigger the destruction of the VPC, cluster, and load balancer.

## Issues and Fixes

- Jenkins build failure due to Docker permissions.
    - Resolution: Modified user permissions and set the Docker socket to allow Jenkins to communicate with the Docker daemon.
- To allow the Jenkins controller to authenticate against agents, the private key permissions were updated using `chmod 400 /var/lib/jenkins/.ssh/id_rsa`.
- Agent labels in `Jenkinsfile` were updated to match the dashboard to prevent deployment issues.
- `load_data.py` script's idempotence was fixed by removing it from the `Dockerfile` to prevent duplicate data loading.

## Optimization
Potential optimizations include:

- Add a bastion host for secure access to instances in private subnets for troubleshooting.
- Implement CloudFront in front of the load balancer to cache static content and boost performance.
- Modularizing Terraform code to reuse and manage components more effectively.
- Implementing Docker layer caching to speed up image builds.
- Auto-scaling ECS tasks based on load for cost-effective performance.
- Utilizing AWS Spot Instances for non-critical parts of the CI/CD pipeline to reduce costs.

Conclusion
The infrastructure implemented for the banking application is secure to an extent, as critical components reside in private subnets, reducing direct exposure to potential attacks. The deployment ensures fault tolerance by leveraging ECS's capability to replace failed containers automatically. In the event of an instance termination, the impact on the application's availability is mitigated by container orchestration in ECS, confirming the resilience of the infrastructure. However, it's noted that all components are in the same region, posing a risk if a regional outage occurs. The containers are deployed within private subnets of the us-east-1a and us-east-1b availability zones, managing ingress and egress through ALB settings for controlled access.

By addressing the stated issues and incorporating suggested optimizations, the infrastructure's reliability, performance, and security will be enhanced for the banking application deployment.





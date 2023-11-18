# Deployment Documentation for ECS Containerized Banking Application

November 4, 2023

By: Khalil Elkharbibi 


## Purpose

The goal of this deployment is to launch a banking application using Amazon's Elastic Container Service (ECS). The previous setup involved a Jenkins agent applying Terraform .tf files to create infrastructure across four public subnets with a synchronized RDS for database consistency. This deployment enhances the existing structure by integrating Docker within ECS for an automated, scalable, and reliable application service delivery. In modern software development, efficient and automated deployment processes are essential for several reasons:

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

**GitHub for Version Control**
- **What**: GitHub is a web-based platform for version control using Git. It allows multiple developers to collaborate on a codebase, track changes, and manage the software development process.
- **Why**: GitHub is widely used for version control because it provides a centralized location for code, making it easy for teams to collaborate. It also offers a range of features like pull requests, code reviews, and issue tracking that enhance the development process.

**Amazon RDS for Database Management**
- **What**: Amazon RDS (Relational Database Service) is a managed database service by AWS. It supports multiple database engines, including MySQL, and simplifies database administration tasks such as provisioning, patching, backup, recovery, and scaling.
- **Why**: Amazon RDS is chosen for its ease of management, automated backups, and scalability. It ensures that the database is reliable and secure, with high availability options.

**Docker for Containerization**
- **What**: Docker is a platform for developing, shipping, and running applications in containers. Containers package an application and its dependencies into a single unit for consistent deployment.
- **Why**: Docker simplifies the deployment process by packaging the application and its dependencies, ensuring that it runs consistently across different environments. It is lightweight, efficient, and allows for efficient scaling of applications.

**Terraform for Infrastructure Provisioning**
- **What**: Terraform is an open-source infrastructure-as-code tool. It enables users to define and provision infrastructure using a declarative configuration language.
- **Why**: Terraform allows for infrastructure provisioning in a consistent and automated manner. It ensures that the infrastructure is versioned, reproducible, and can be scaled as needed.

**Jenkins for Automation**
- **What**: Jenkins is an open-source automation server that helps automate various parts of the software development process, including building, testing, and deploying code.
- **Why**: Jenkins streamlines the CI/CD process by automating tasks, like building Docker images, running tests, and deploying to ECS. It ensures reliability, consistency, and rapid development.

**ECS for Container Orchestration**
- **What**: Amazon Elastic Container Service (ECS) is a fully managed container orchestration service that simplifies the deployment, management, and scaling of containerized applications using Docker containers.
- **Why**: ECS is chosen for its ability to manage and scale Docker containers seamlessly. It ensures high availability and fault tolerance while simplifying the deployment and management of containers.

These technologies were selected to streamline the deployment process, ensuring agility, security, and scalability while maintaining consistency and predictability in the development and deployment of the containerized banking application.

## System Diagram
![Deployment 7 drawio](https://github.com/kha1i1e/Deployment_7/assets/140761974/c708a5e8-71dc-40fc-b8bf-33023bc71317)


## Steps

**VPC Configuration for ECS**
> - The `vpc.tf` file generates a VPC with:
>  - 2 Availability Zones equipped with both public and private subnets for failover resilience.
>  - The private subnets host the Fargate tasks containing the application in Docker containers.
>  - The public subnets facilitate internet traffic routing to the private ones, mediated by an ALB.
>  - A Security Group configured to permit traffic on port 8000 for private subnet communication, and port 80 for ALB internet access.
>  - Two route tables directing internet-bound traffic through their respective internet and NAT gateways.
>  - A NAT Gateway for outbound internet connectivity from private subnets, safeguarding against direct inbound connections.

```
resource "aws_vpc" "app_vpc" {
  cidr_block = "172.28.0.0/16"
}

resource "aws_subnet" "public_a" {
  vpc_id            = aws_vpc.app_vpc.id
  cidr_block        = "172.28.0.0/18"
  availability_zone = "us-east-1a"

  tags = {
    "Name" = "public | us-east-1a"
  }
}

resource "aws_subnet" "private_a" {
  vpc_id            = aws_vpc.app_vpc.id
  cidr_block        = "172.28.64.0/18"
  availability_zone = "us-east-1a"

  tags = {
    "Name" = "private | us-east-1a"
  }
}
```

**ECS Architecture**
> - The `main.tf` script articulates:
>  - An ECS Cluster housing Fargate instances as secure hosts for the Dockerized Flask app.
>  - An ECS Task Definition detailing the app container setup, including image, ports, and computing resources needed.
>  - An ECS Service specifying the task quantities and load balancing configurations, ensuring service continuity and scalability.
>  - A CloudWatch Log Group for logging and monitoring purposes, essential for scaling and load balancing policy formulation.
```
provider "aws" {
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  region     = "us-east-1"

}

# Cluster
resource "aws_ecs_cluster" "aws-ecs-cluster" {
  name = "urlapp-cluster"
  tags = {
    Name = "url-ecs"
  }
}

resource "aws_cloudwatch_log_group" "log-group" {
  name = "/ecs/bank-logs"

  tags = {
    Application = "bank-app"
  }
}

# Task Definition

resource "aws_ecs_task_definition" "aws-ecs-task" {
  family = "url-task"

  container_definitions = <<EOF
  [
  {
      "name": "url-container",
      "image": "kha1i1e/dep7bankapp:latest",
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/bank-logs",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      },
      "portMappings": [
        {
          "containerPort": 5000
        }
      ]
    }
  ]
  EOF

  requires_compatibilities = ["FARGATE"]
  network_mode             = "awsvpc"
  memory                   = "1024"
  cpu                      = "512"
  execution_role_arn       = "arn:aws:iam::156156311593:role/ecsTaskExecutionRole"
  task_role_arn            = "arn:aws:iam::156156311593:role/ecsTaskExecutionRole"

}

# ECS Service
resource "aws_ecs_service" "aws-ecs-service" {
  name                 = "url-ecs-service"
  cluster              = aws_ecs_cluster.aws-ecs-cluster.id
  task_definition      = aws_ecs_task_definition.aws-ecs-task.arn
  launch_type          = "FARGATE"
  scheduling_strategy  = "REPLICA"
  desired_count        = 2
  force_new_deployment = true

  network_configuration {
    subnets = [
      aws_subnet.private_a.id,
      aws_subnet.private_b.id
    ]
    assign_public_ip = false
    security_groups  = [aws_security_group.ingress_app.id]
  }

  load_balancer {
    target_group_arn = aws_lb_target_group.bank-app.arn
    container_name   = "url-container"
    container_port   = 5000
  }

}

```

**Terraform File Adjustments for ecs-vpc**
> - To adapt the `main.tf` for different deployments:
>  - Assign your ECS Cluster a specific name.
>  - Customize the task definition with relevant container naming and image referencing conventions from Docker Hub.
>  - Configure the ECS Service and associate it with the appropriate load balancer.
>  - Establish IAM roles for task execution and task definition.
    
![Deployment 7 ecs](https://github.com/kha1i1e/Deployment_7/assets/140761974/178f7c45-5231-4c9e-b44b-6dfbd59b4259)

**ALB Configuration via Terraform**
> - The `alb.tf` manages ALB setup to secure and direct traffic:
>  - Construct an ALB to interface with the Fargate-hosted containers.
>  - Define listener rules to reroute traffic to the desired target group.

``` # Target Group
resource "aws_lb_target_group" "bank-app" {
  name        = "dep7-bankapp-app"
  port        = 8000
  protocol    = "HTTP"
  target_type = "ip"
  vpc_id      = aws_vpc.app_vpc.id

  health_check {
    enabled = true
    path    = "/health"
  }

  depends_on = [aws_alb.bank_app]
}

# Application Load Balancer
resource "aws_alb" "bank_app" {
  name               = "dep7-bankapp-lb"
  internal           = false
  load_balancer_type = "application"

  subnets = [
    aws_subnet.public_a.id,
    aws_subnet.public_b.id,
  ]

  security_groups = [
    aws_security_group.http.id,
  ]

  depends_on = [aws_internet_gateway.igw]
}

resource "aws_alb_listener" "bank_app_listener" {
  load_balancer_arn = aws_alb.bank_app.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.bank-app.arn
  }
}

output "alb_url" {
  value = "http://${aws_alb.bank_app.dns_name}"
}
    
```
![Deployment 7 load balancer](https://github.com/kha1i1e/Deployment_7/assets/140761974/51f14bc9-079e-4b2b-9d19-5e4b90cbf10a)

**GitHub Workflow and Security**
> - Jenkins automation ties into GitHub through webhooks, initiating pipelines upon code changes.
> - GitHub token generation for Jenkins integration involves:
>  - Accessing GitHub settings to generate personal access tokens with comprehensive repository and admin hook permissions.
    

**Branch Strategy for Repository Management**
> - Branch management incorporates a multi-branch pipeline Jenkins infrastructure, container hosting setup, and application feature development.

**Docker Image Build and Test**
> - Utilize the `Dockerfile` at the repository root to assemble the Flask application image.
> - Test Docker image build processes locally before pushing to Docker Hub with provided Docker commands, ensuring local functionality at `localhost:8000`.
  
 ``` # Update dockerfile
FROM python:3.7

RUN git clone https://github.com/kha1i1e/Deployment_7.git

WORKDIR Deployment_7

RUN pip install pip --upgrade

RUN pip install -r requirements.txt

RUN pip install mysqlclient

RUN pip install gunicorn

RUN python database.py

EXPOSE 8000

ENTRYPOINT python -m gunicorn app:app -b 0.0.0.0

```

**Securing Docker Hub Integration with Jenkins**
> - Configure Docker Hub access by creating and securely storing a token, then integrate this with Jenkins for Docker image pushes.

**Jenkins Setup and Pipeline Execution**
> - The Jenkins ecosystem, formed within an existing VPC, uses provided Terraform scripts and consists of a trio of EC2 instances for the Jenkins setup.
> - Configure the Jenkins master via shell scripts and EC2 resources, setting up SSH keys and installing necessary plugins for pipeline activities.

**Credential Management for AWS and Docker Hub**
> - Add AWS and Docker Hub credentials in Jenkins for Terraform actions and Docker image deployment, using the 'Add Credentials' feature in Jenkins settings.

# Jenkins Deploy Agent

> The Jenkins deploy agent sets up the VPC for the ECS cluster, with an application load balancer configured using the `jenkins.sh` script. This script is passed to Terraform within the EC2 resource block to set up the Jenkins agent service and Terraform.

## Controller and Agent Communication

To set up and configure `awsDeploy` and `awsDeploy2` agents for communication with the Jenkins controller, follow these steps:

> 1. Ensure the Jenkins user on the Jenkins controller has the appropriate private key at `/var/lib/jenkins/.ssh/id_rsa`.
> 2. On the Jenkins dashboard, navigate to `Manage Jenkins > Manage Nodes and Clouds > New Node`.
> 3. Name the node, select 'Permanent Agent', and click 'OK'.
> 4. Specify the 'Remote root directory' for the agent, such as `/home/ubuntu/agent1`.
> 5. In the 'Labels' field, add a label like `awsDeploy` to identify the agent.
> 6. For 'Launch method', choose 'Launch agent via SSH'.
> 7. Add the private IP address of the agent server in the 'Host' field.
> 8. Select the Jenkins credentials in the 'Credentials' dropdown.
> 9. Enter the agent server's username (e.g., `ubuntu`) in the 'Username' field.
> 10. In the 'Private Key' section, choose 'Enter directly' and paste the private key from the Jenkins server.
> 11. Save the configuration and launch the agent.
```
# Jenkins-Agent Infrastructure
git add jenkinsTerraform
# Create files main.tf, terraform.tfvars, variables.tf, installfile1.sh, installfile2.sh, installfile3.sh
terraform init
terraform validate
terraform plan
terraform apply
# After creation of the Jenkins Agent infrastructure
git add main.tf terraform.tfvars variables.tf installfile1.sh installfile2.sh installfile3.sh
git commit -a
# Make a file .gitignore and put all the names of the files for Git to ignore
git push --set-upstream origin second
git switch main
git merge second
git push --all

# initTerraform
git switch second
# Update .tf files
git commit -a
git switch main
get merge second
git push --all
```
![Deployment 7 Jenkins](https://github.com/kha1i1e/Deployment_7/assets/140761974/d80b30a5-9020-4808-acdb-d4db4c2ca340)

## MySQL Database in AWS RDS

To share data across application instances, use AWS RDS to create a MySQL database:

> 1. Go to AWS RDS on the AWS console and click 'Create database'.
> 2. Choose 'MySQL' and the 'Free tier' option.
> 3. Under 'Settings', fill in the 'DB instance identifier', 'Master username', and 'Master password'.
> 4. Ensure 'Public access' is set to 'Yes' and configure the security group for port 3306 ingress (egress to all traffic).
> 5. In 'Additional configuration', name your initial database (e.g., 'banking') and opt-out of encryption.
> 6. After creating the database, find it under 'Databases' and click on its name to get the endpoint from 'Connectivity & security'.


![DB Endpoint](https://github.com/kha1i1e/Deployment_7/assets/140761974/9f5d70a0-fb18-495f-b92e-4aa3c7191771)

## Application Code

Update the application code to connect to the new database:

> 1. In `app.py`, `database.py`, and `load_data.py`, find the `DATABASE_URL` constant.
> 2. Replace it with the new RDS connection string format: `[mysql+pymysql://]{username}:{password}@{host}/{database-name}`.
```
# update datapoints

git clone https://github.com/kha1i1e/Deployment_7.git
cd Deployment_7/
git init
git branch second
git switch second
# Update Database_URL in app.py, database.py, and load_data.py
git commit -a
```
![Deployment 7 RDS](https://github.com/kha1i1e/Deployment_7/assets/140761974/95e2709d-5935-40f0-a6ef-2ff3fa1e2e64)

## Deploy

Deploy the application with these git commands:

```bash
git add .
git commit -m 
git push
```
![Deployment_7 banking app deployed](https://github.com/kha1i1e/Deployment_7/assets/140761974/60a33a35-e629-4f09-aa90-4329b1deaf06)

After pushing changes, the `main.tf` file's output includes the load balancer's URL for application access.

## Destroy

To remove resources:

> 1. Uncomment the `destroy` stage in the `Jenkinsfile`.
> 2. Commit and push the changes to trigger the destruction of the VPC, cluster, and load balancer.

## Issues and Fixes

> - Jenkins build failure due to Docker permissions.
>    - Resolution: Modified user permissions and set the Docker socket to allow Jenkins to communicate with the Docker daemon.
> - To allow the Jenkins controller to authenticate against agents, the private key permissions were updated using `chmod 400 /var/lib/jenkins/.ssh/id_rsa`.
> - Agent labels in `Jenkinsfile` were updated to match the dashboard to prevent deployment issues.
> - `load_data.py` script's idempotence was fixed by removing it from the `Dockerfile` to prevent duplicate data loading.

## Optimization
Potential optimizations include:

> - Add a bastion host for secure access to instances in private subnets for troubleshooting.
> - Implement CloudFront in front of the load balancer to cache static content and boost performance.
> - Modularizing Terraform code to reuse and manage components more effectively.
> - Implementing Docker layer caching to speed up image builds.
> - Auto-scaling ECS tasks based on load for cost-effective performance.
> - Utilizing AWS Spot Instances for non-critical parts of the CI/CD pipeline to reduce costs.

## Conclusion

The infrastructure implemented for the banking application is secure to an extent, as critical components reside in private subnets, reducing direct exposure to potential attacks. The deployment ensures fault tolerance by leveraging ECS's capability to replace failed containers automatically. In the event of an instance termination, the impact on the application's availability is mitigated by container orchestration in ECS, confirming the resilience of the infrastructure. However, it's noted that all components are in the same region, posing a risk if a regional outage occurs. The containers are deployed within private subnets of the us-east-1a and us-east-1b availability zones, managing ingress and egress through ALB settings for controlled access.

By addressing the stated issues and incorporating suggested optimizations, the infrastructure's reliability, performance, and security will be enhanced for the banking application deployment.





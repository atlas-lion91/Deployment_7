# Deploying Banking Application in Elastic Container Service (ECS)

November 4, 2023

By: Khalil Elkharbibi

## Purpose

The purpose of this documentation is to outline the steps and processes involved in deploying a Banking Application in Elastic Container Service (ECS) using various DevOps tools and technologies. In modern software development, efficient and automated deployment processes are essential for several reasons:

1. **Agility and Rapid Development:** Rapid application development is a cornerstone of today's software industry. An efficient deployment process allows developers to focus on writing code while the automated pipeline takes care of building, testing, and deploying the application.

2. **Consistency and Predictability:** Manual deployments often lead to inconsistencies and unpredictable outcomes. An automated process ensures that each deployment is consistent, reducing errors and minimizing disruptions.

3. **Resource Efficiency:** Traditional infrastructure provisioning can be time-consuming and resource-intensive. With infrastructure-as-code tools like Terraform, we can provision and manage cloud resources more efficiently, enabling rapid scaling as needed.

4. **Security and Reliability:** Security is paramount in any deployment. With proper configurations, we can ensure that sensitive data is protected, and the application remains reliable.

5. **Scaling and Fault Tolerance:** Modern applications must be able to scale and recover from failures seamlessly. Using containerization and container orchestration, ECS provides the foundation for scalable, fault-tolerant deployments.

This deployment approach leverages a range of tools and services, including GitHub for version control, Amazon RDS for database management, Docker for containerization, Terraform for infrastructure provisioning, Jenkins for automation, and ECS for container orchestration. Each step in this documentation contributes to achieving these essential DevOps goals, enabling a streamlined and secure deployment process.


## Steps

### Step 1: Create a Dockerfile

**What it is:** A Dockerfile is a script that contains a set of instructions for building a Docker image. It specifies the base image, sets up the application environment, and defines how the application should run.

**Why we used it:** The Dockerfile allows us to containerize the Banking Application, ensuring consistent and reproducible deployments. Docker containers are lightweight and portable, making it easy to run applications in various environments.

**How to do it:** Create a Dockerfile like the one below and place it in the project directory:

```Dockerfile
# Dockerfile for the Banking Application
FROM python:3.9
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "app.py"]
```

### Step 2: Design VPC Infrastructure and CI/CD Pipeline

**What it is:** Diagram the Virtual Private Cloud (VPC) infrastructure and Continuous Integration/Continuous Deployment (CI/CD) pipeline.

**Why we're doing it:** Designing your VPC infrastructure and CI/CD pipeline helps visualize and plan your deployment architecture.

**How to do it:** Refer to this [AWS documentation](https://docs.aws.amazon.com/prescriptive-guidance/latest/load-balancer-stickiness/subnets-routing.html) for VPC diagram guidelines.

### Step 3: GitHub/Git

**What it is:** Set up a GitHub repository for Jenkins integration.

**Why we're doing it:** GitHub serves as the repository from which Jenkins retrieves files to build, test, and deploy the banking application. It is essential for version control and collaboration.

**How to do it:**  
1. Create a GitHub repository.
2. Generate a token from GitHub.
3. Provide the token to your Jenkins EC2 instance.




### Step 4: Amazon's Relational Database Service (RDS)

**What it is:** Configure Amazon's Relational Database Service (RDS) to manage the MySQL database.

**Why we're doing it:** RDS automates backups and synchronizes data across regions, availability zones, and instances. It ensures security and reliability for your database.

**How to do it:** Update the Database URL in the following files: `app.py`, `database.py`, and `load_data.py` with the necessary RDS configurations.


### Step 5: Terraform

**What it is:** Use Terraform to create infrastructure for Jenkins agents, application deployment, and networking.

**Why we're doing it:** Terraform enables automated provisioning and management of resources, ensuring infrastructure consistency and scalability.

**How to do it:**  
1. Create Terraform scripts for Jenkins agent infrastructure, networking, and the application.
2. Deploy the infrastructure using Terraform.

### Step 6: Jenkins

**What it is:** Automate the build, test, and deployment of the Banking Application using Jenkins.

**Why we're doing it:** Jenkins automates the software development process and ensures that the application is built, tested, and deployed efficiently.

**How to do it:**  
1. Set up Jenkins and Jenkins nodes.
2. Create a Key Pair.
3. Configure Jenkins.
4. Configure Jenkins nodes, AWS access keys, Docker credentials, and install necessary plugins.

### Step 7: Use Jenkins Terraform Agent

**What it is:** Execute Terraform scripts using Jenkins to create the Banking Application infrastructure and deploy the application on ECS with Application Load Balancer.

**Why we're doing it:** Jenkins simplifies the deployment process by executing Terraform scripts to create infrastructure and deploy the application.

**How to do it:**  
1. Create a Jenkins build named "Deployment_7".
2. Run the Jenkinsfile from the GitHub Repository: https://github.com/kha1i1e/deployment_7.git.

### Step 8: Application Load Balancer (ALB)

**What it is:** Create an Application Load Balancer to distribute incoming web traffic to application instances.

**Why we're doing it:** The ALB ensures the application remains available, responsive, and efficient by distributing traffic evenly and redirecting it in case of server issues.

**How to do it:**  
Refer to the provided Terraform code for creating an ALB.

### Step 9: Main Infrastructure (main.tf)

**What it is:** Define the main infrastructure using Terraform, including the ECS cluster, task definition, and ECS service.

**Why we're doing it:** Terraform helps define the desired state of your infrastructure and manage the resources to match that configuration, ensuring scalability and consistency.

**How to do it:**  
Refer to the provided Terraform code (main.tf) to define and deploy the main infrastructure.

### Conclusion

**Why we used ECS:** We used ECS for its ability to simplify containerized application deployment and management.

**Why we used RDS:** RDS automates database management, ensuring data synchronization, security, and reliability.

**Why we used Terraform:** Terraform enables automation, scalability, and consistency in managing infrastructure resources.

**Why we used Jenkins:** Jenkins automates the software development process, including building, testing, and deploying applications.

This comprehensive documentation provides a step-by-step guide to deploying your banking application in an efficient and scalable manner. Use the provided code snippets and instructions to facilitate a smooth deployment process.

This deployment approach leverages a range of tools and services, including GitHub for version control, Amazon RDS for database management, Docker for containerization, Terraform for infrastructure provisioning, Jenkins for automation, and ECS for container orchestration. Each step in this documentation contributes to achieving these essential DevOps goals, enabling a streamlined and secure deployment process.

We'll delve into each step, explaining **what it is** and **why we used it**.

## Steps

### Step 1: Create a Dockerfile

**What it is:** A Dockerfile is a script that contains a set of instructions for building a Docker image. It specifies the base image, sets up the application environment, and defines how the application should run.

**Why we used it:** The Dockerfile allows us to containerize the Banking Application, ensuring consistent and reproducible deployments. Docker containers are lightweight and portable, making it easy to run applications in various environments.

```Dockerfile
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

### Step 2: Update Resource Names in main.tf and ALB.tf

**What it is:** In the main.tf and ALB.tf Terraform files, we define AWS resources, such as VPC configurations, security groups, and load balancers. In this step, we update resource names and settings as needed.

**Why we used it:** Properly naming and configuring resources in Terraform ensures clarity and consistency. The main.tf file is critical for setting up the infrastructure, while ALB.tf deals with configuring the Application Load Balancer, which is essential for directing traffic to our ECS instances.

```
# main.tf
resource "aws_instance" "jenkins_manager" {
  ami           = "ami-0123456789"
  instance_type = "t2.micro"
  tags = {
    Name = "jenkins-manager"
  }
  # ... other configurations
}
```

```
# ALB.tf
resource "aws_lb_target_group" "bank_app" {
  name        = "bank-app-target-group"
  port        = 5000
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
}
```

### Step 3: Use Terraform to Create Jenkins Manager and Agents

**What it is:** Terraform is an infrastructure-as-code tool that automates the provisioning and management of cloud resources. In this step, we use Terraform to create the Jenkins Manager and Agents infrastructure.

**Why we used it:** Terraform ensures that our Jenkins infrastructure is created consistently and can be easily scaled. By defining infrastructure as code, we can manage our Jenkins servers and agents efficiently.

```
# Jenkins.tf
resource "aws_instance" "jenkins_manager" {
  ami           = "ami-0123456789"
  instance_type = "t2.micro"
  tags = {
    Name = "jenkins-manager"
  }
  # ... other configurations
}
```

### Step 4: Observe VPC.tf for Network Configuration

**What it is:** The VPC.tf Terraform file defines the Virtual Private Cloud (VPC) configuration, including subnets, route tables, and security groups.

**Why we used it:** Proper network configuration is vital for isolating components, securing traffic, and enabling communication between resources. By observing VPC.tf, we ensure that our infrastructure is well-architected and follows best practices.

```
# VPC.tf
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
  enable_dns_support = true
  enable_dns_hostnames = true
  tags = {
    Name = "main-vpc"
  }
  # ... other configurations
}
```

### Step 5: Create an RDS Database

**What it is:** Amazon's Relational Database Service (RDS) is a fully managed database service that simplifies database management tasks, such as setup, patching, and scaling. In this step, we create an RDS database to store application data.

**Why we used it:** An RDS database provides a reliable and scalable data store for our application. It offers automated backups, data synchronization across regions, and high security standards. Our Banking Application relies on this database for data integrity and availability.

```
# RDS.tf
resource "aws_db_instance" "bank_app_db" {
  allocated_storage    = 20
  storage_type         = "gp2"
  engine               = "mysql"
  engine_version       = "5.7"
  instance_class       = "db.t2.micro"
  name                 = "bankingdb"
  username             = "admin"
  password             = "securepassword"
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
  tags = {
    Name = "bank-app-db"
  }
  # ... other configurations
}
```

### Step 6: Create a Docker Image

**What it is:** A Docker image is a template of an application with all the dependencies it needs to run. A Dockerfile contains instructions to build this image.

**Why we used it:** Creating a Docker image allows us to package the Banking Application along with its dependencies, ensuring consistency across different environments. It simplifies deployment and enhances scalability.

### Step 7: Create Jenkins Manager and Agents

**What it is:** Jenkins is an open-source automation server that helps automate various parts of the software development process, including building, testing, and deploying applications.

**Why we used it:** Jenkins is crucial for automating the build, test, and deployment processes. The Jenkins Manager and Agents infrastructure ensures that tasks are distributed efficiently, and the pipeline runs smoothly.

### Step 8: Create

 an ECS Cluster

**What it is:** Amazon Elastic Container Service (ECS) is a managed container orchestration service that simplifies the deployment, management, and scaling of containerized applications.

**Why we used it:** ECS enables us to run containers in a scalable and reliable manner. It automates container management, making it easier to deploy applications. The ECS cluster serves as the foundation for hosting our application containers.

### Step 9: Deploy the Banking Application to ECS

**What it is:** This step involves deploying the Banking Application containers to the ECS cluster created in the previous step.

**Why we used it:** Deploying to ECS is a scalable and reliable approach. ECS manages the deployment of containers, ensuring they are highly available and can be easily scaled based on demand.

### Step 10: Set Up an Application Load Balancer (ALB)

**What it is:** An Application Load Balancer (ALB) is a service that evenly distributes incoming web traffic to multiple ECS instances, ensuring high availability and improved performance.

**Why we used it:** The ALB helps manage traffic efficiently. It distributes incoming requests to available instances, avoiding overloads and ensuring continuous availability.

### Step 11: Set Up Jenkins for Automation

**What it is:** Jenkins automates the build, test, and deployment of the Banking Application. It requires the proper installation of Jenkins, Java, and necessary plugins.

**Why we used it:** Jenkins simplifies the automation of DevOps processes. It orchestrates the deployment pipeline, ensuring that the application is built, tested, and deployed consistently.

### Step 12: Configure Jenkins for the Pipeline

**What it is:** In this step, we configure Jenkins by creating key pairs, setting up Jenkins nodes, configuring AWS access and secret keys, and Docker credentials.

**Why we used it:** Proper Jenkins configuration ensures that the pipeline operates smoothly. Access keys, credentials, and node settings are essential for automating tasks within the deployment process.

### Step 13: Use Jenkins for Terraform Script Execution

**What it is:** Jenkins is used to execute Terraform scripts to create the Banking Application infrastructure and deploy the application on ECS with an Application Load Balancer.

**Why we used it:** Jenkins orchestrates the deployment pipeline. In this step, it executes Terraform scripts, automating the provisioning and deployment of infrastructure.

### Step 14: Monitor and Troubleshoot

This step involves monitoring the deployed infrastructure, logs, and application performance. It allows for troubleshooting and optimization.

Monitoring ensures that the infrastructure remains secure and performs well. It helps identify and resolve issues promptly.

### Step 15: Enhance for Optimization

This step involves optimizing the deployment process. Possible enhancements include improving automation, enhancing security, and adding content delivery network (CDN) for static content.

 Optimization is an ongoing process. Enhancing automation and security while adding a CDN can further improve deployment efficiency and the application's performance.


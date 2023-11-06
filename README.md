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
      "image": "tsanderson77/bankapp11:latest",
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

```
# Target Group
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

### Step 3: Use Terraform to Create Jenkins Manager and Agents

**What it is:** Terraform is an infrastructure-as-code tool that automates the provisioning and management of cloud resources. In this step, we use Terraform to create the Jenkins Manager and Agents infrastructure.

**Why we used it:** Terraform ensures that our Jenkins infrastructure is created consistently and can be easily scaled. By defining infrastructure as code, we can manage our Jenkins servers and agents efficiently.

```
################################### A W S #################################

provider "aws" {
  access_key = var.access_key  #enter your aws access_key
  secret_key = var.secret_key  #enter your aws secret_key
  region = var.region   #Availability Zone
  #profile = "Admin"
}

################################### I N S T A N C E # 1 #################################

resource "aws_instance" "tf_local_instance_1" {
  ami = var.ami                            
  instance_type = var.instance_type
  subnet_id = var.instance_1_attach_existing_subnet 
  vpc_security_group_ids = var.instance_1_existing_sg #[aws_security_group.tf_local_security_group_1.id]  #new sg and exiting
  user_data = "${file(var.instance_1_installs)}"
  key_name = var.key_pair          
  associate_public_ip_address = true  

  tags = {
    "Name" : var.aws_instance_1_name     
  }
}

################################### I N S T A N C E # 2 #################################

resource "aws_instance" "tf_local_instance_2" {
  ami = var.ami                            #AMI ID for Ubuntu
  instance_type = var.instance_type
  subnet_id = var.instance_2_attach_existing_subnet
  
  #security_groups = var.instance_1_existing_sg   
  vpc_security_group_ids = var.instance_1_existing_sg #[aws_security_group.tf_local_security_group_2.id]   
  user_data = "${file(var.instance_1_installs)}"
  key_name = var.key_pair          # name of your SSH key pair
  associate_public_ip_address = true  # Enable Auto-assign public IP

  tags = {
    "Name" : var.aws_instance_2_name   #name of the instance in AWS
  }
}

################################### I N S T A N C E # 3 #################################

resource "aws_instance" "tf_local_instance_3" {
  ami = var.ami                         
  instance_type = var.instance_type
  subnet_id = var.instance_3_attach_existing_subnet
  vpc_security_group_ids = var.instance_1_existing_sg #[aws_security_group.tf_local_security_group_2.id]   
  user_data = "${file(var.instance_1_installs)}"
  key_name = var.key_pair          # name of your SSH key pair
  associate_public_ip_address = true  # Enable Auto-assign public IP

  tags = {
    "Name" : var.aws_instance_3_name   #name of the instance in AWS
  }
}
################################### O U T P U T #################################

output "instance_1_ip" {            
  value = aws_instance.tf_local_instance_1.public_ip
}
```

### Step 4: Observe VPC.tf for Network Configuration

**What it is:** The VPC.tf Terraform file defines the Virtual Private Cloud (VPC) configuration, including subnets, route tables, and security groups.

**Why we used it:** Proper network configuration is vital for isolating components, securing traffic, and enabling communication between resources. By observing VPC.tf, we ensure that our infrastructure is well-architected and follows best practices.

```
# VPC.tf
resource "aws_vpc" "app_vpc" {
  cidr_block = "172.28.0.0/16"
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
  password             = ""
  parameter_group_name = "default.mysql5.7"
  skip_final_snapshot  = true
  tags = {
    Name = "bank-app-db"
  }
}
```

```
# git update datapoints
git clone https://github.com/kha1i1e/Deployment_7.git
cd Deployment_7/
git init
git branch second
git switch second
# Update Database_URL in app.py, database.py, and load_data.py
git commit -a
```

### Step 6: Create a Docker Image

**What it is:** A Docker image is a template of an application with all the dependencies it needs to run. A Dockerfile contains instructions to build this image.

**Why we used it:** Creating a Docker image allows us to package the Banking Application along with its dependencies, ensuring consistency across different environments. It simplifies deployment and enhances scalability.

```
# GIT - docker file

# Update dockerfile
git commit -a # added comment
# Test the docker file
# Build the image
docker build -t bankapp
# Create the container and deploy the application
docker container run -p 80:8000 bankapp
# Retag an image to include the Docker Hub account name
docker tag bankapp kha1i1e/banking
# Login into Docker Hub on your terminal
docker login # enter credentials
# Push the image to Docker Hub
docker push kha1i1e/bankapp
# Delete image from Docker Hub
```


### Step 7: Create Jenkins Manager and Agents

**What it is:** Jenkins is an open-source automation server that helps automate various parts of the software development process, including building, testing, and deploying applications.

**Why we used it:** Jenkins is crucial for automating the build, test, and deployment processes. The Jenkins Manager and Agents infrastructure ensures that tasks are distributed efficiently, and the pipeline runs smoothly.

```
#Jenkins-Agent Infrastructure

git add jenkinsTerraform
# Create files main.tf, terraform.tfvars, variables.tf, installfile1.sh, installfile2.sh, installs3.sh
terraform init
terraform validate
terraform plan
terraform apply
# After creation of the Jenkins Agent infrastructure
git add main.tf terraform.tfvars variables.tf installfile1.sh installfile2.sh installs3.sh
git commit -a
# Make a file .gitignore and put all the names of the files for Git to ignore
git push --set-upstream origin second
git switch main
git merge second
git push --all
```


### Step 8: Create an ECS Cluster

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

**Successfully Deployment Banking Application**:
![Deployment_7 banking app deployed](https://github.com/kha1i1e/Deployment_7/assets/140761974/f76f3e47-34b1-46c7-82f2-5ea9d86f6d98)



### Step 14: Monitor and Troubleshoot

This step involves monitoring the deployed infrastructure, logs, and application performance. It allows for troubleshooting and optimization.

Monitoring ensures that the infrastructure remains secure and performs well. It helps identify and resolve issues promptly.

### Step 15: Enhance for Optimization

This step involves optimizing the deployment process. Possible enhancements include improving automation, enhancing security, and adding content delivery network (CDN) for static content.

 Optimization is an ongoing process. Enhancing automation and security while adding a CDN can further improve deployment efficiency and the application's performance.


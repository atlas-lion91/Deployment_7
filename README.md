# Banking App Deployment Documentation

## Purpose

The purpose of this documentation is to outline the deployment process for the Banking App, demonstrating the deployment of ECS infrastructure using Terraform. This deployment aims to:

- Deploy a Jenkins infrastructure comprising Main, Docker, and Agent servers.
- Utilize Jenkins agents to deploy the Banking Flask application to ECS using Terraform and Docker.

## Steps

### Step 1: Dockerize the Banking App

**Why**: Containerizing the application ensures consistency and isolation.

1. Create a Dockerfile for the Banking App and place it in your repository. Ensure it is connected to the RDS database.

### Step 2: Terraform Configuration

**Why**: Terraform automates infrastructure provisioning and management.

2. Modify the following resources in `main.tf` and `ALB.tf`:
   - Cluster name
   - Task Definition: Family
   - Container definitions (name, image, containerPort)
   - execution_role_arn
   - task_role_arn
   - ECS Service name
   - container_name
   - container_port

3. Configure an AWS instance (Instance 2) with Terraform and default-jre.

4. Clone the Kura repository to the Jenkins instance and push it to a new repository.

5. Create a Jenkins agent on the second instance.

6. Configure AWS credentials in Jenkins.

7. Place your Terraform files and user data script in the `initTerraform` directory.

### Step 3: Create VPCs with Terraform

**Why**: Practice creating AWS infrastructure and Git operations.

8. Create two VPCs using Terraform, one in US-east-1 and the other in US-west-2. Each VPC should have:
   - 2 Availability Zones
   - 2 Public Subnets
   - 2 EC2 instances
   - 1 Route Table
   - Security Group Ports: 8000, 22

9. Create an RDS database to link application databases and create the second tier.

10. Branch, update, and merge MySQL endpoint changes in your repository.

### Step 6: Create Jenkins Multibranch Pipeline

**Why**: Jenkins automates the build and deployment process.

11. Create a Jenkins multibranch pipeline to implement different Jenkinsfiles for various branches of the project.

### Step 7: Check Infrastructures and Applications

12. Review application and infrastructure status for both US-east-1 and US-west-2.

### Step 8: Create an Application Load Balancer

**Why**: Load balancers distribute traffic for better availability.

13. Create an application load balancer for both US-east-1 and US-west-2.

### Step 9: Consider Additional Infrastructure

**Why**: Enhance security and availability.

14. Consider adding the following to the infrastructures:
   - Reverse web proxy like nginx
   - Private subnets
   - NAT Gateway
   - API Gateway
   - Network Load Balancer

## Issues/Troubleshooting

- AMI and Key Pair not working in Terraform when creating the US-west-2 infrastructure. Resolution steps included modifying the AMI and creating a key pair for the correct region.
- Testing deployment of the application using the user data script had issues resolved by adding the command to activate the environment.
- Test phase failure in Jenkins build due to an unknown database issue. Resolution involved correcting the database name in the URL.
- Application load balancer test issues were resolved by configuring the correct VPC and security group.

## Conclusion

The deployment process successfully demonstrated infrastructure setup using Terraform, and automation with Jenkins for application deployment. Automation enhanced efficiency, but further improvements can be made, such as optimizing Jenkins agent usage. Potential enhancements include using separate agents for testing and deployment tasks.

This deployment serves as a valuable practice for DevOps engineers in automating infrastructure and application deployment, and offers opportunities for ongoing optimization.

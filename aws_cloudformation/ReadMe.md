# Chatwoot Deployment on AWS with GitHub CI/CD
This guide provides a comprehensive walkthrough on deploying Chatwoot on AWS using GitHub CI/CD workflows. The deployment involves setting up an infrastructure with AWS services using CloudFormation scripts and managing different environments (development and production) with GitHub Actions.

## Introduction
Chatwoot is an open-source customer engagement suite. This guide will help you deploy Chatwoot on AWS, leveraging managed services like RDS, Elasticache, and ALB, and automate the deployment process using GitHub Actions.

## Prerequisites
- An active AWS account
- Domain names for your Chatwoot environments (`dev-chat.sumoscheduler.com` for development and `chat.sumoscheduler.com` for production)
- Access to GitHub repository to manage CI/CD workflows

## Architecture
This deployment follows a three-tier architecture:
- **Network Tier**: VPC, Subnets, NAT Gateway, Internet Gateway
- **Application Tier**: EC2 Instances behind an Application Load Balancer
- **Data Tier**: RDS for PostgreSQL, Elasticache for Redis

## AWS Setup
### Network Configuration
1. **VPC**: Create a Virtual Private Cloud named `chatwoot-vpc` with CIDR block `10.0.0.0/16`.
2. **Subnets**: Configure public and private subnets across two availability zones.
3. **Internet & NAT Gateways**: Set up to facilitate internet access for public and private subnets respectively.
4. **Route Tables**: Define routes for internet access and subnet associations.

### Application Load Balancer (ALB)
Configure an ALB to handle traffic and distribute it across Chatwoot instances.

### Database and Cache
1. **RDS for PostgreSQL**: Set up a multi-AZ RDS instance with a dedicated security group.
2. **Elasticache for Redis**: Create a Redis cluster with multi-AZ configuration.

### Bastion Hosts
Deploy bastion servers in public subnets to securely access Chatwoot instances in private subnets.

## GitHub CI/CD Workflows
### Repository Secrets
Ensure the following secrets are set in your GitHub repository:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`
- `AWS_REGION`
- `ACM_CERTIFICATE_ARN`: Obtained from AWS Certificate Manager for `*.sumoscheduler.com`
- `KEY_NAME`: Key pair name from EC2
- `RDS_MASTER_USERNAME` and `RDS_MASTER_PASSWORD`
- `SSH_KEY`: Contents of your PEM file for SSH access
- `ROUTE_HZ_ID`: Hosted zone ID from Route 53 for `sumoscheduler.com`

### Workflows
You have two workflows for different environments, each executing CloudFormation scripts:

1. **Development Workflow**: Deploys infrastructure for `dev-chat.sumoscheduler.com`.
2. **Production Workflow**: Deploys infrastructure for `chat.sumoscheduler.com`.

Each workflow includes:
- **network.yml**: Sets up VPC, subnets, gateways, and route tables.
- **application_load_balancer.yml**: Configures ALB.
- **postgresql.yml**: Provisions RDS instance.
- **redis.yml**: Sets up Elasticache.
- **bastion.yml**: Deploys bastion hosts.
- **chatwoot.yml**: Installs and configures Chatwoot.
- **auto-scaling.yml**: Manages auto-scaling policies for EC2 instances.

## Deployment Steps
1. **Set Up AWS Infrastructure**: Run the CloudFormation scripts using the GitHub Actions workflows. Ensure all necessary AWS resources are created and configured.
2. **Install Chatwoot**: SSH into the Chatwoot instance via Bastion Host and execute installation scripts.
3. **Configure Environment Variables**: Update the .env file with RDS and Elasticache credentials.
4. **Run Database Migrations**: Prepare the database using Rails commands.
5. **Verify Deployment**: Access the application via the domain names to ensure correct setup and functionality.
6. **Create AMI and ASG**: If everything is working, create an AMI for Chatwoot and configure the Auto Scaling Group to manage instance scaling.

## Verification and Maintenance
- Regularly monitor and update the CloudFormation scripts and CI/CD workflows.
- Ensure all AWS resources are properly secured and optimized for cost and performance.
- Keep GitHub secrets up to date and manage permissions carefully.

By following this guide, you’ll set up a robust, scalable Chatwoot deployment on AWS, fully integrated with GitHub’s CI/CD capabilities to streamline updates and maintenance.
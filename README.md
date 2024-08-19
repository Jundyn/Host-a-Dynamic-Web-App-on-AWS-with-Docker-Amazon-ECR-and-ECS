# Deploying a Dynamic Web Application on AWS with Docker, Amazon ECR, and Amazon ECS

This project demonstrates the deployment of a dynamic web application on AWS using Docker, Amazon ECR (Elastic Container Registry), and Amazon ECS (Elastic Container Service). The deployment leverages a 3-tier architecture with public and private subnets across two availability zones to ensure high availability and fault tolerance.

## Project Overview

The architecture comprises:
![Alt text](https://github.com/Jundyn/Host-a-Dynamic-Web-App-on-AWS-with-Docker-Amazon-ECR-and-ECS/blob/main/How_to_Host_a_Dynamic_Web_App_on_AWS_with_Docker_Amazon_ECR_and_Amazon_ECS.jpg)

- **Docker**: For containerizing the application.
- **Amazon ECR**: To store Docker images.
- **Amazon ECS**: For container orchestration.
- **Amazon RDS**: To manage the relational database.
- **ECS Fargate**: For serverless compute to run containers.
- **Application Load Balancer**: To distribute incoming application traffic.
- **Auto Scaling**: For dynamic scaling of ECS tasks.
- **Route 53**: For domain name registration and DNS management.
- **AWS S3**: For storing environment variables.
- **IAM Roles**: For managing permissions for ECS tasks.
- **Bastion Host**: For secure SSH access.
- **Security Groups**: For network access control.
- **Certificate Manager**: For managing SSL/TLS certificates.

The application is designed to run within a Virtual Private Cloud (VPC) with both public and private subnets, providing a secure and scalable environment.

## Tools & Technologies

- **Docker**: Containerization of the application.
- **Git**: Version control for source code.
- **GitHub**: Repository hosting for Dockerfile and application code.
- **AWS CLI**: Command-line interface for managing AWS services.
- **Flyway**: Database schema migration tool.
- **Visual Studio Code**: IDE for script development.
- **Amazon ECR**: Docker image storage.
- **Amazon ECS**: Container orchestration on AWS.
- **VPC**: Virtual network setup.
- **Amazon RDS**: Relational database service.
- **ECS Fargate**: Serverless compute for containers.
- **Application Load Balancer**: Load distribution.
- **Auto Scaling**: Dynamic scaling of ECS tasks.
- **Route 53**: Domain name registration and DNS management.
- **AWS S3**: Storage for environment variables.
- **IAM Roles**: Permissions management for ECS tasks.
- **Bastion Host**: Secure SSH access.
- **Security Groups**: Network access control.
- **Certificate Manager**: SSL/TLS certificate management.

## Prerequisites

Before you start, ensure you have the following tools installed:

- **Git**: [Download Git](https://git-scm.com/downloads)
- **Visual Studio Code**: [Download Visual Studio Code](https://code.visualstudio.com/)
- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- **Flyway**: [Download Flyway](https://flywaydb.org/download)

## Deployment Guide

1. **Clone the Repository**
   ```bash
   git clone https://github.com/your-repo/your-project.git
   cd your-project
   ```

2. **Create Dockerfile for the dynamic web app:**
```bash
   # Use the latest version of the Amazon Linux base image
FROM amazonlinux:2

# Update all installed packages to their latest versions
RUN yum update -y 

# Install necessary packages
RUN yum install -y unzip wget httpd git

# Install PHP and MySQL
RUN amazon-linux-extras enable php7.4 && yum install -y \
  php php-common php-pear php-cgi php-curl php-mbstring php-gd php-mysqlnd \
  php-gettext php-json php-xml php-fpm php-intl php-zip mysql-community-server

# Configure MySQL
RUN wget https://repo.mysql.com/mysql80-community-release-el7-3.noarch.rpm && \
    rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022 && \
    yum localinstall mysql80-community-release-el7-3.noarch.rpm -y && \
    yum install mysql-community-server -y

# Clone and prepare the application
WORKDIR /var/www/html
ARG PERSONAL_ACCESS_TOKEN
ARG GITHUB_USERNAME
ARG REPOSITORY_NAME
ARG WEB_FILE_ZIP
ARG WEB_FILE_UNZIP
ARG DOMAIN_NAME
ARG RDS_ENDPOINT
ARG RDS_DB_NAME
ARG RDS_MASTER_USERNAME
ARG RDS_DB_PASSWORD
ENV PERSONAL_ACCESS_TOKEN=$PERSONAL_ACCESS_TOKEN \
    GITHUB_USERNAME=$GITHUB_USERNAME \
    REPOSITORY_NAME=$REPOSITORY_NAME \
    WEB_FILE_ZIP=$WEB_FILE_ZIP \
    WEB_FILE_UNZIP=$WEB_FILE_UNZIP \
    DOMAIN_NAME=$DOMAIN_NAME \
    RDS_ENDPOINT=$RDS_ENDPOINT \
    RDS_DB_NAME=$RDS_DB_NAME \
    RDS_MASTER_USERNAME=$RDS_MASTER_USERNAME \
    RDS_DB_PASSWORD=$RDS_DB_PASSWORD

RUN git clone https://$PERSONAL_ACCESS_TOKEN@github.com/$GITHUB_USERNAME/$REPOSITORY_NAME.git && \
    unzip $REPOSITORY_NAME/$WEB_FILE_ZIP -d $REPOSITORY_NAME/ && \
    cp -av $REPOSITORY_NAME/$WEB_FILE_UNZIP/. /var/www/html && \
    rm -rf $REPOSITORY_NAME && \
    sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf && \
    chmod -R 777 /var/www/html && \
    chmod -R 777 storage/ && \
    sed -i '/^APP_ENV=/ s/=.*$/=production/' .env && \
    sed -i "/^APP_URL=/ s/=.*$/=https:\/\/$DOMAIN_NAME\//" .env && \
    sed -i "/^DB_HOST=/ s/=.*$/=$RDS_ENDPOINT/" .env && \
    sed -i "/^DB_DATABASE=/ s/=.*$/=$RDS_DB_NAME/" .env && \
    sed -i "/^DB_USERNAME=/ s/=.*$/=$RDS_MASTER_USERNAME/" .env && \
    sed -i "/^DB_PASSWORD=/ s/=.*$/=$RDS_DB_PASSWORD/" .env

COPY AppServiceProvider.php app/Providers/AppServiceProvider.php
EXPOSE 80 3306
ENTRYPOINT ["/usr/sbin/httpd", "-D", "FOREGROUND"]
```
3. **Build Docker Image:**
```bash
docker build --build-arg PERSONAL_ACCESS_TOKEN=<your_token> \
              --build-arg GITHUB_USERNAME=<your_username> \
              --build-arg REPOSITORY_NAME=<your_repo> \
              --build-arg WEB_FILE_ZIP=<your_zip> \
              --build-arg WEB_FILE_UNZIP=<your_unzip> \
              --build-arg DOMAIN_NAME=<your_domain> \
              --build-arg RDS_ENDPOINT=<rds_endpoint> \
              --build-arg RDS_DB_NAME=<rds_db_name> \
              --build-arg RDS_MASTER_USERNAME=<rds_username> \
              --build-arg RDS_DB_PASSWORD=<rds_password> \
              -t <image-tag> .
```
4. **Push Docker Image to Amazon ECR:**
```bash
# Create ECR repository
aws ecr create-repository --repository-name <repository-name> --region <region>
# Login to ECR and push Docker image
aws ecr get-login-password | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
docker tag <image-tag> <repository-uri>
docker push <repository-uri>
``` 

5. **Configure AWS Services**
   - **Set up VPC**, **Subnets**, **Security Groups**, and **IAM Roles**.
   - **Create an Amazon RDS instance** for the database.
   - **Deploy the ECS service** using the Docker image from ECR.
   - **Configure Application Load Balancer** and **Auto Scaling**.

6. **DNS and SSL/TLS Configuration**
   - **Register a domain with Route 53** and set up DNS records.
   - **Use Certificate Manager** to manage SSL/TLS certificates and configure HTTPS for the Load Balancer.

7. **Set Environment Variables**
   ```bash
   aws s3 cp s3://your-bucket-name/env-vars.json .
   ```

8. **Run Database Migrations**
   ```bash
   flyway -url=jdbc:postgresql://<db-endpoint>:5432/your-db -user=your-username -password=your-password migrate
   ```

9. **Access Your Application**
   - Use the domain name registered with Route 53 to access your application via the Application Load Balancer.
## Scripts

Deployment scripts and configurations are provided in the GitHub repository. Ensure to review and adjust them according to your environment and requirements.

## Web Page

The deployed web page is available in the ![Alt text](https://github.com/Jundyn/Host-a-Dynamic-Web-App-on-AWS-with-Docker-Amazon-ECR-and-ECS/blob/main/rentzone-webpage.png)

## Additional Resources

Here are some useful resources that can help you understand and customize the deployment process further:

- **[AWS Documentation](https://docs.aws.amazon.com/)**
- **[Docker Documentation](https://docs.docker.com/)**
- **[GitHub Documentation](https://docs.github.com/)**
- **[Flyway Documentation](https://flywaydb.org/documentation/)**

Feel free to customize the scripts and configurations according to your specific needs. This README is designed to provide a comprehensive guide that showcases the skills and processes involved in this DevOps project.


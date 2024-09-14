# Deploying a Dynamic Web Application on AWS with Docker, Amazon ECR, and Amazon ECS

This project demonstrates the deployment of a dynamic web application on AWS using Docker, Amazon ECR (Elastic Container Registry), and Amazon ECS (Elastic Container Service). The deployment leverages a 3-tier architecture with public and private subnets across two availability zones to ensure high availability and fault tolerance.

## Project Overview

The architecture comprises:
![Alt text](https://github.com/Jundyn/Host-a-Dynamic-Web-App-on-AWS-with-Docker-Amazon-ECR-and-ECS/blob/main/How_to_Host_a_Dynamic_Web_App_on_AWS_with_Docker_Amazon_ECR_and_Amazon_ECS.jpg)

## Prerequisites

Before you start, ensure you have the following tools installed:

- **Git**: [Download Git](https://git-scm.com/downloads)
- **Visual Studio Code**: [Download Visual Studio Code](https://code.visualstudio.com/)
- **Docker**: [Install Docker](https://docs.docker.com/get-docker/)
- **AWS CLI**: [Install AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html)
- **Flyway**: [Download Flyway](https://flywaydb.org/download)

1. **Virtual Private Cloud (VPC)**: 
   - **Description**: A 3-Tier VPC is established with Public Subnets, Private App Subnets, and Private Data Subnets across 2 availability zones. This segregation enhances security and organizes different components.

2. **Public Subnets**:
   - **Purpose**: Host infrastructure components such as the NAT Gateway and Application Load Balancer.

3. **Internet Gateway**:
   - **Purpose**: Enables communication between instances in the VPC and the internet.

4. **Private Subnets**:
   - **Purpose**: Host the web server designed to serve web pages and applications securely over the internet.

5. **EC2 Instances**:
   - **Usage**: Host the WordPress website, accessible via an EC2 Instance Connect Endpoint.

6. **Bastion Host**:
   - **Purpose**: Assists in migrating data into the RDS database while providing perimeter access and control security.

7. **AWS Fargate**:
   - **Purpose**: Allows running containers without managing servers or clusters.

8. **S3 Bucket**:
   - **Purpose**: Provides environmental file storage.

9. **Application Load Balancer**:
   - **Purpose**: Distributes web traffic across an Auto Scaling Group of EC2 instances in two availability zones to ensure high availability and fault tolerance.

10. **Availability Zones**:
    - **Purpose**: Ensures high availability and fault tolerance by deploying resources across multiple zones.

11. **Resources**:
    - **Description**: NAT Gateway, Bastion Host, and Application Load Balancer are deployed in Public Subnets.

12. **Auto Scaling Group**:
    - **Purpose**: Dynamically manages EC2 instances to ensure scalability, fault tolerance, and elasticity.

13. **Route 53**:
    - **Purpose**: Manages domain name registration and DNS records.

14. **Docker**:
    - **Purpose**: Dockerfile is used to build Docker images with Build Arguments and Environment Variables to handle secrets securely. This setup allows local image building and pushing to Amazon ECR.

15. **Security Groups**:
    - **Purpose**: Acts as a network firewall to control traffic.

16. **Instances**:
    - **Access**: Configured to access the internet via the NAT Gateway, even when in private subnets.

17. **GitHub**:
    - **Purpose**: Utilized for version control and collaboration, storing web files.

18. **Git**:
    - **Purpose**: Manages a `.gitignore` file to prevent the Dockerfile from being committed to GitHub.

19. **Certificate Manager**:
    - **Purpose**: Manages SSL/TLS certificates for secure application communications.

20. **SNS (Simple Notification Service)**:
    - **Purpose**: Configured to alert about activities within the Auto Scaling Group.

21. **EFS (Elastic File System)**:
    - **Purpose**: Provides a shared file system.

22. **RDS (Relational Database Service)**:
    - **Purpose**: Manages the database.

23. **IAM Roles**:
    - **Purpose**: Allows for authentication with AWS using secret access keys and access key IDs to push the container image to ECR.

24. **Flyway**:
    - **Purpose**: Organizes and securely migrates database scripts via SSH Tunnel into the MySQL RDS.

## Deployment Steps

### VPC Setup

1. **Create a VPC**:
   - Establish a VPC with public and private subnets across two availability zones.
   - Enable DNS hostnames within the VPC.

2. **Set Up Internet Gateway**:
   - Attach the Internet Gateway to the VPC.

3. **Create Subnets**:
   - Define public and private subnets for the availability zones.
   - Enable auto-assign IP settings for the public subnets.

4. **Create Route Table**:
   - Add a route to direct network traffic to the Internet Gateway.
   - Associate the route table with the subnets.

5. **Create NAT Gateway**:
   - Deploy a NAT Gateway in the public subnet for internet access from private subnets.

### Security and Gateway Configuration

1. **Configure Security Groups**:
   - Allow necessary inbound and outbound traffic.

2. **Set Up Route 53**:
   - Manage domain name registration and DNS records.

3. **Use AWS Certificate Manager**:
   - Manage SSL/TLS certificates for secure communication.

### Script Configuration

1. **Create Personal Access Token**:
   - Docker will use this token to clone the Application Code repository when building the Docker image.

2. **Set Up Project Folder**:
   - Create a folder in Visual Studio Code to host necessary files such as Dockerfile and AppServiceProvider.php.

3. **Build Dockerfile**:
   - Create a Dockerfile to build the Docker image, updating values and information as needed.

4. **Replace AppServiceProvider.php**:
   - Enter the script into the new AppServiceProvider.php file to ensure proper redirection from HTTP to HTTPS.

5. **Manage Sensitive Information**:
   - Rename Dockerfile to `Dockerfile-reference` and create a `.gitignore` file to prevent committing sensitive information to GitHub.

6. **Create New Dockerfile**:
   - Create a new Dockerfile in the `rentzone` folder, incorporating Build Arguments and Environment Variables.

7. **Build Docker Image**:
   - Write a script (`build_image.sh`) to build the Docker image, setting values for Build Arguments.

8. **Make Shell Script Executable**:
   - Run `chmod +x build_image.sh` to make the shell script executable.

9. **Execute Build Script**:
   - Run the `build_image.sh` script to build the Docker image via Visual Studio Code’s integrated terminal.

## Install AWS Command Line Interface (CLI)

1. Visit [AWS CLI Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).
2. Follow the instructions specific to your operating system to install the AWS CLI via the command line.

## IAM User Creation

1. Create an IAM user with administrative access. Generate a secret access key and access key ID for this user to authenticate with AWS and push container images to ECR.
2. Run the following command in Command Prompt or Terminal to configure the AWS CLI with your IAM user credentials:
   ```bash
   aws configure
   ```
   Enter the Access Key ID and Secret Access Key when prompted.

## ECR and Application Setup

1. Create a repository in Amazon ECR using the AWS CLI:
   ```bash
   aws ecr create-repository --repository-name <repository-name> --region <region>
   ```

2. Tag and push the Docker image to the ECR repository. Replace placeholders with the actual tag name and repository URI:
   ```bash
   docker tag <image-tag> <repository-uri>
   docker push <repository-uri>
   ```

3. Log in to Amazon ECR and push the Docker image:
   ```bash
   aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<region>.amazonaws.com
   docker push <repository-uri>
   ```

## Key Pair Creation

1. Create a Key Pair for SSH access to the management instance.
2. Download the Key Pair `.pem` file and move it to your PowerShell or terminal directory to simplify SSH commands.

## Bastion Host Configuration

1. Launch a Bastion Host instance using the Amazon Linux 2 AMI and T2 Micro instance type.
2. Attach the Key Pair created earlier and select the appropriate VPC.
3. Choose the Public AZ1 Subnet and assign the Bastion Host Security Group.

## Flyway Configuration

1. Download Flyway Community Version and open the Flyway folder in Visual Studio Code.
2. Under the `conf` directory, create a `flyway.conf` file with the following configuration:
   ```properties
   flyway.url=jdbc:mysql://localhost:3306/
   flyway.user=
   flyway.password=
   flyway.locations=filesystem:sql
   flyway.cleanDisabled=false
   ```

3. Update the `flyway.conf` file with your RDS configuration details.
4. Add SQL scripts to the `sql` directory within the Flyway folder.
5. Rename the SQL script to the format `version__description.sql` for Flyway compatibility.

## SSH Tunnel Setup

1. To set up an SSH tunnel, use the following command based on your operating system:
   - **PowerShell**:
     ```powershell
     ssh -i <key_pair.pem> ec2-user@<public-ip> -L 3306:<rds-endpoint>:3306 -N
     ```
   - **Linux/macOS**:
     ```bash
     ssh -i "YOUR_EC2_KEY" -L LOCAL_PORT:RDS_ENDPOINT:REMOTE_PORT EC2_USER@EC2_HOST -N -f
     ```

2. Open a terminal, navigate to where your Key Pair is stored, and execute the command.
3. In the Flyway directory, run the Flyway migration command:
   ```bash
   ./flyway migrate
   ```

## Create Application Load Balancer

1. First, create a Target Group with IPv4 address type and HTTP on port 80.
2. Create the Application Load Balancer in the VPC, selecting Public Subnet AZ1 and AZ2. Use the Application Load Balancer Security Group created earlier.

## Certificate Manager

1. Request an SSL certificate from AWS Certificate Manager, using DNS validation.
2. Once requested, create a Route 53 record set to validate domain ownership.

## HTTPS Listener Configuration

1. Create an HTTPS Listener for the Application Load Balancer using the SSL certificate.
2. Configure the Listener to use port 443.

## Environment File Creation

1. Create an environment file named `rentzone.env` to store Docker environment variables:
   ```env
   PERSONAL_ACCESS_TOKEN=
   GITHUB_USERNAME=
   REPOSITORY_NAME=
   WEB_FILE_ZIP=
   WEB_FILE_UNZIP=
   DOMAIN_NAME=
   RDS_ENDPOINT=
   RDS_DB_NAME=
   RDS_MASTER_USERNAME=
   RDS_DB_PASSWORD=
   ```

2. Add `rentzone.env` to the `.gitignore` file to prevent it from being tracked by Git.

## AWS S3 Bucket Creation

1. Create an S3 Bucket in the same region as your VPC.
2. Upload the `rentzone.env` file to the S3 Bucket.

## 2nd IAM User Creation

1. Create an IAM Role for ECS with permissions to access the S3 bucket.
2. Assign the role to ECS with inline policies for `S3:GetObject` and `S3:GetBucketLocation`.

## ECS Cluster Creation

1. Create an ECS Cluster and assign the VPC.
2. Select Private App Subnet AZ1 and AZ2.

## Task Definition Creation

1. Create a Task Definition specifying CPU, memory, and IAM role.
2. Retrieve the image URI from the ECR repository.
3. Configure environment variables using the S3 bucket ARN.

## ECS Service Creation

1. **Create the ECS Service:**
   - Select the existing ECS cluster you created.
   - Choose 'Use custom' and select the Task Definition created earlier under Application Type.

2. **Configure Desired Tasks:**
   - Set the number of desired tasks to 2.

3. **Configure Networking:**
   - Ensure the VPC environment is selected.
   - Enable Private App Subnet AZ1 and AZ2.

4. **Select Security Group:**
   - Choose the existing Security Group and remove the default Security Group.

5. **Public IP Settings:**
   - Turn off Public IP settings as the container is in a private subnet.

6. **Load Balancing:**
   - Select the existing Application Load Balancer.

7. **Listener Configuration:**
   - Choose Port 80 HTTP as the existing Listener.

8. **Target Group:**
   - Select the existing Target Group.

9. **Service Auto Scaling:**
   - Set Auto Scaling policies for minimum and maximum tasks:
     - Minimum number of tasks: 1
     - Maximum number of tasks: 4
   - Configure Policy Name, Target Value, Scale-out cooldown period, and Scale-in cooldown period.

10. **Verify Service:**
    - After a few minutes, check the ECS Service in the ECS console. Under the Health and Metrics tab, you should see 2 healthy targets running.


## Record Set Route 53 Creation

1. **Create a Record Set:**
   - Go to the Route 53 hosted zone.
   - Select the domain name and set the record type to 'A - record type.'
   - Set the Record name to 'www.'

2. **Route Traffic:**
   - Route traffic to 'Alias to Application and Classic Load Balancer.'
   - Select the region where the Application Load Balancer was created.
   - Choose the Application Load Balancer.

## How to Use

1. **Clone the Repository:**
   ```bash
   git clone <repository-url>
   ```

2. **Create Required Resources:**
   - Follow AWS documentation to set up the necessary resources such as VPC, subnets, Internet Gateway, etc.

3. **Set Up the Application:**
   - Use the provided scripts to deploy the WordPress application on EC2 instances within the VPC.

4. **Configure Services:**
   - Set up Auto Scaling Group, Load Balancer, and other services according to the architecture.

5. **Access the Application:**
   - Access the WordPress website via the Load Balancer's DNS name.

## Additional Resources

- **AWS Documentation:** Detailed guides on setting up VPC, EC2, Auto Scaling, Load Balancer, and other services.
- **GitHub Repository Files:** Access repository files for scripts, architectural diagrams, and configuration files at [https://github.com/Jundyn/Host-a-Dynamic-Web-App-on-AWS-with-Docker-Amazon-ECR-and-ECS](<repository-url>).

## Contributing

Contributions are welcome! Please fork the repository and submit a pull request with your enhancements.

## Problems Solved

1. **Scalable and Reliable Infrastructure:** Automated Docker image builds and deployments to AWS ECS, reducing deployment time and minimizing human error.
2. **Secure and Efficient Image Management:** Used AWS ECR for secure Docker image management, addressing version control and security.
3. **Database Management:** Utilized AWS RDS for MySQL, including read replicas for load distribution and effective database management.
4. **High Availability:** Ensured application availability by deploying across two Availability Zones and using Elastic Load Balancing.
5. **Cost Management:** Optimized costs through AWS Cost Explorer, AWS Budgets, and Reserved Instances.

## Issues Faced and Resolutions

- **AWS Authentication and Permissions:** Managed IAM roles and permissions carefully to avoid access errors. Verified and set up IAM policies with the least privilege principle.
- **Increasing Costs:** Mitigated costs by turning off unused resources and updating Dockerfile with new Elastic IP and RDS Endpoint information.
- **Docker Image Management:** Addressed image creation, tagging, and pushing challenges with careful management.
- **Network Configuration:** Resolved networking issues by double-checking configurations and ensuring correct setup of VPCs, Security Groups, and Load Balancers.

## Lessons Learned

- **Docker Image Management:** Automated build and push processes using CI/CD pipelines to ensure consistency and reduce human error.
- **Network Configuration:** Utilized Terraform and AWS CloudFormation for standardizing and automating network resource creation.
- **AWS Linux and Bash:** Enhanced skills in AWS Linux and bash scripting through practical use in Visual Studio Code.
- **Networking:** Gained knowledge in VPC and NAT Gateway configuration for internet access and resource connectivity.
- **IAM Management Best Practices:** Avoided using root accounts for cloud creation, enforced MFA, and regularly updated root account security.
- **Dockerfile Best Practices:** Created a `.gitignore` file to prevent sensitive information from being committed to GitHub.
- **AWS ECS and Docker Integration:** Improved understanding of container orchestration and management with AWS ECS and Docker.

---

## Useable Scripts 

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

5. **Set Environment Variables**
   ```bash
   aws s3 cp s3://your-bucket-name/env-vars.json .
   ```

6. **Run Database Migrations**
   ```bash
   flyway -url=jdbc:postgresql://<db-endpoint>:5432/your-db -user=your-username -password=your-password migrate
   ```
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


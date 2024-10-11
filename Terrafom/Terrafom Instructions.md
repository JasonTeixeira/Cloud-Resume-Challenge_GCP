## **Step-by-Step Instructional Breakdown for the Terraform Project**

This step-by-step guide will walk you through how to use the provided Terraform configuration file to set up a highly available web application on AWS using CloudFront, RDS MySQL, EC2, S3, and related services. Below, I will break down each phase of the process, explaining what’s happening and how each resource works within the architecture.

### **main.tf** (Terraform Configuration)

```
hcl
Copy code
provider "aws" {
  region = "us-east-1"  # Change the region if needed
}

# VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  enable_dns_support   = true
  enable_dns_hostnames = true
  tags = {
    Name = "high-availability-vpc"
  }
}

# Public Subnets
resource "aws_subnet" "public_subnet_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.1.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
  tags = {
    Name = "public-subnet-1"
  }
}

resource "aws_subnet" "public_subnet_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.2.0/24"
  availability_zone = "us-east-1b"
  map_public_ip_on_launch = true
  tags = {
    Name = "public-subnet-2"
  }
}

# Private Subnets
resource "aws_subnet" "private_subnet_1" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.3.0/24"
  availability_zone = "us-east-1a"
  tags = {
    Name = "private-subnet-1"
  }
}

resource "aws_subnet" "private_subnet_2" {
  vpc_id            = aws_vpc.main.id
  cidr_block        = "10.0.4.0/24"
  availability_zone = "us-east-1b"
  tags = {
    Name = "private-subnet-2"
  }
}

# Internet Gateway
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id
  tags = {
    Name = "main-igw"
  }
}

# NAT Gateway (For private subnets)
resource "aws_eip" "nat_eip" {
  vpc = true
}

resource "aws_nat_gateway" "nat_gateway" {
  allocation_id = aws_eip.nat_eip.id
  subnet_id     = aws_subnet.public_subnet_1.id
}

# Route Tables
resource "aws_route_table" "public_rt" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.igw.id
  }
}

resource "aws_route_table_association" "public_rt_assoc_1" {
  subnet_id      = aws_subnet.public_subnet_1.id
  route_table_id = aws_route_table.public_rt.id
}

resource "aws_route_table_association" "public_rt_assoc_2" {
  subnet_id      = aws_subnet.public_subnet_2.id
  route_table_id = aws_route_table.public_rt.id
}

# Security Groups
resource "aws_security_group" "alb_sg" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_security_group" "ec2_sg" {
  vpc_id = aws_vpc.main.id
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    security_groups = [aws_security_group.alb_sg.id]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Load Balancer (ALB)
resource "aws_lb" "app_lb" {
  name               = "app-lb"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.alb_sg.id]
  subnets            = [aws_subnet.public_subnet_1.id, aws_subnet.public_subnet_2.id]

  enable_deletion_protection = false
}

resource "aws_lb_target_group" "app_tg" {
  name        = "app-tg"
  port        = 80
  protocol    = "HTTP"
  vpc_id      = aws_vpc.main.id
  target_type = "instance"
}

resource "aws_lb_listener" "app_listener" {
  load_balancer_arn = aws_lb.app_lb.arn
  port              = "80"
  protocol          = "HTTP"

  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.app_tg.arn
  }
}

# EC2 Instances
resource "aws_instance" "web_instance_1" {
  ami                         = "ami-0c55b159cbfafe1f0"  # Amazon Linux 2 AMI ID
  instance_type               = "t3.micro"
  subnet_id                   = aws_subnet.private_subnet_1.id
  security_groups             = [aws_security_group.ec2_sg.id]
  associate_public_ip_address = false

  tags = {
    Name = "WebServer-1"
  }
}

resource "aws_instance" "web_instance_2" {
  ami                         = "ami-0c55b159cbfafe1f0"
  instance_type               = "t3.micro"
  subnet_id                   = aws_subnet.private_subnet_2.id
  security_groups             = [aws_security_group.ec2_sg.id]
  associate_public_ip_address = false

  tags = {
    Name = "WebServer-2"
  }
}

# Auto Scaling Group and Launch Configuration (optional)
resource "aws_launch_configuration" "app_launch_config" {
  image_id      = "ami-0c55b159cbfafe1f0"
  instance_type = "t3.micro"
  security_groups = [aws_security_group.ec2_sg.id]
}

resource "aws_autoscaling_group" "app_asg" {
  vpc_zone_identifier         = [aws_subnet.private_subnet_1.id, aws_subnet.private_subnet_2.id]
  launch_configuration        = aws_launch_configuration.app_launch_config.id
  min_size                    = 1
  max_size                    = 3
  desired_capacity            = 1

  tag {
    key                 = "Name"
    value               = "AutoScalingWebApp"
    propagate_at_launch = true
  }
}

# RDS MySQL Database
resource "aws_db_instance" "db_instance" {
  allocated_storage    = 20
  engine               = "mysql"
  instance_class       = "db.t3.micro"
  name                 = "mydb"
  username             = "admin"
  password             = "password"
  multi_az             = true
  publicly_accessible  = false
  skip_final_snapshot  = true
  db_subnet_group_name = aws_db_subnet_group.db_subnet_group.name

  tags = {
    Name = "RDS-MySQL"
  }
}

resource "aws_db_subnet_group" "db_subnet_group" {
  name       = "my-db-subnet-group"
  subnet_ids = [aws_subnet.private_subnet_1.id, aws_subnet.private_subnet_2.id]
}

# S3 Bucket for static content
resource "aws_s3_bucket" "app_bucket" {
  bucket = "high-availability-webapp-bucket"
  acl    = "public-read"

  versioning {
    enabled = true
  }

  tags = {
    Name = "StaticContentBucket"
  }
}

# CloudFront Distribution
resource "aws_cloudfront_distribution" "cdn" {
  origin {
    domain_name = aws_s3_bucket.app_bucket.bucket_regional_domain_name
    origin_id   = "S3-static-content"

    s3_origin_config {
      origin_access_identity = aws_cloudfront_origin_access_identity.origin_identity.cloudfront_access_identity_path
    }
  }

  enabled             = true
  is_ipv6_enabled     = true
  default_root_object = "index.html"

  default_cache_behavior {
    target_origin_id       = "S3-static-content"
    viewer_protocol_policy = "redirect-to-https"
    allowed_methods        = ["GET", "HEAD"]
    cached_methods         = ["GET", "HEAD"]

    forwarded_values {
      query_string = false
      cookies {
        forward = "none"
      }
    }
  }

  restrictions {
    geo_restriction {
      restriction_type = "none"
    }
  }

  viewer_certificate {
    cloudfront_default_certificate = true
  }

  tags = {
    Name = "CloudFront-CDN"
  }
}

resource "aws_cloudfront_origin_access_identity" "origin_identity" {
  comment = "Origin access identity for S3 bucket"
}

```

### **variables.tf** (Optional)

```
hcl
Copy code
variable "region" {
  default = "us-east-1"
}

variable "instance_type" {
  default = "t3.micro"
}

variable "db_password" {
  type      = string
  default   = "password"
}

```

### **outputs.tf** (Outputs for information)**

```
hcl
Copy code
output "load_balancer_dns" {
  value = aws_lb.app_lb.dns_name
}

output "s3_bucket_name" {
  value = aws_s3_bucket.app_bucket.bucket
}

output "cloudfront_distribution" {
  value = aws_cloudfront_distribution.cdn.domain_name
}

output "db_endpoint" {
  value = aws_db_instance.db_instance.endpoint
}

```

### **How This Works**:

1. **VPC and Subnets**: The project creates a new VPC with public and private subnets across two availability zones, ensuring high availability.
2. **Internet Gateway and NAT Gateway**: Provides internet access for public subnets and allows private subnets to reach the internet.
3. **EC2 Instances**: Configures EC2 instances for web servers, which are distributed across private subnets.
4. **Load Balancer**: An Application Load Balancer (ALB) distributes traffic to EC2 instances.
5. **Auto Scaling**: Optional auto-scaling configuration for EC2 instances.
6. **RDS MySQL**: Creates a highly available RDS MySQL instance with multi-AZ failover.
7. **S3 and CloudFront**: Creates an S3 bucket for static assets and integrates it with CloudFront for global content delivery.
8. **Security**: Implements security groups to control access to EC2, ALB, and other resources.

This Terraform file should be executed with the standard Terraform commands:

```bash
bash
Copy code
terraform init
terraform plan
terraform apply

```

Be sure to configure your AWS CLI or environment with the necessary permissions before applying the infrastructure. You can customize the resources, regions, instance types, or scaling policies according to your needs.

---

### **1. Prerequisites**

Before we begin, you need to ensure the following:

- **AWS Account**: You'll need an AWS account with sufficient permissions to create resources (e.g., VPC, EC2, RDS, S3).
- **AWS CLI**: Install and configure the AWS CLI with your credentials:
    
    ```bash
    bash
    Copy code
    aws configure
    
    ```
    
    Enter your AWS access key, secret key, region, and default output format.
    
- **Terraform Installed**: Download and install Terraform.
    
    ```bash
    bash
    Copy code
    terraform -v
    
    ```
    
    Ensure it's properly installed by checking the version.
    
- **IAM Role/Permissions**: Make sure your user has necessary AWS IAM roles/permissions to manage EC2, RDS, VPC, S3, CloudFront, etc.

---

### **2. Clone or Create a New Terraform Directory**

Set up a new directory for your project:

```bash
bash
Copy code
mkdir aws-web-app-terraform
cd aws-web-app-terraform

```

Place the `main.tf`, `variables.tf`, and `outputs.tf` files into this directory, or clone the files if they exist in a repository.

---

### **3. Initialize Terraform**

Once you have the configuration files in place, you need to initialize Terraform. This step ensures Terraform downloads the appropriate provider plugins (in this case, AWS) needed to interact with the AWS infrastructure.

```bash
bash
Copy code
terraform init

```

**What’s happening here?**

- Terraform will download the necessary provider for AWS (AWS SDK) and prepare the environment for the rest of the workflow.

---

### **4. Review the Terraform Plan**

Before applying the infrastructure, it’s important to see what Terraform will do. This allows you to review all the actions Terraform will take, and ensure you’re comfortable with the resources it plans to create.

```bash
bash
Copy code
terraform plan

```

**What’s happening here?**

- Terraform is checking your configuration files and previewing the actions it will take to create the AWS infrastructure.
- It will display all the resources it plans to create, such as VPC, EC2 instances, RDS, S3, and more.

---

### **5. Apply the Terraform Configuration**

After reviewing the plan, you can now apply the configuration. This will create all the necessary infrastructure components on AWS.

```bash
bash
Copy code
terraform apply

```

Terraform will display the resources to be created and ask for confirmation to proceed. Type `yes` to confirm and begin provisioning.

**What’s happening here?**

- Terraform will create all the resources defined in the `main.tf` file on AWS. This includes:
    - **VPC and Subnets**: Creates a Virtual Private Cloud (VPC) and sets up public and private subnets across two Availability Zones.
    - **Internet Gateway and NAT Gateway**: The Internet Gateway allows public access, and the NAT Gateway enables private subnets to reach the internet.
    - **Security Groups**: Establishes security rules to control access to EC2, ALB, and other resources.
    - **Application Load Balancer (ALB)**: This balances incoming traffic across multiple EC2 instances.
    - **EC2 Instances**: Creates web server instances in private subnets.
    - **RDS MySQL**: Creates a MySQL database instance with multi-AZ failover for high availability.
    - **S3 Bucket and CloudFront**: S3 stores static content, and CloudFront delivers this content globally via its CDN (Content Delivery Network).

---

### **6. Check Outputs**

Once the resources are successfully provisioned, Terraform will output critical information such as the Load Balancer DNS, S3 bucket name, and CloudFront distribution. This is defined in the `outputs.tf` file.

```bash
bash
Copy code
Outputs:

load_balancer_dns         = "app-lb-123456789.us-east-1.elb.amazonaws.com"
s3_bucket_name            = "high-availability-webapp-bucket"
cloudfront_distribution   = "abc123.cloudfront.net"
db_endpoint               = "mydb.123456789012.us-east-1.rds.amazonaws.com"

```

**What’s happening here?**

- Terraform is displaying important information about the resources it created, such as:
    - **Load Balancer DNS**: The domain where your application is hosted.
    - **S3 Bucket Name**: The bucket where your static content is stored.
    - **CloudFront Distribution**: The domain for your CDN distribution to deliver static content globally.
    - **RDS Endpoint**: The database connection endpoint for your MySQL database.

---

### **7. Access Your Web Application**

Now, you can access your web application by visiting the Load Balancer’s DNS:

```bash
bash
Copy code
http://<load_balancer_dns>

```

You can also test your CloudFront distribution:

```bash
bash
Copy code
http://<cloudfront_distribution>

```

This URL will serve static content stored in the S3 bucket through the globally distributed CloudFront CDN.

---

### **8. Modify Infrastructure (Optional)**

If you need to make changes to the infrastructure (e.g., update instance sizes, adjust scaling policies, etc.), modify the respective resources in the `main.tf` file.

Once the changes are made, you can apply them with:

```bash
bash
Copy code
terraform apply

```

Terraform will intelligently update only the resources that need modification without disrupting others.

---

### **9. Destroy the Infrastructure (Optional)**

When you no longer need the infrastructure, you can clean it up to avoid incurring unnecessary costs by destroying the resources.

```bash
bash
Copy code
terraform destroy

```

Terraform will list all the resources it plans to destroy and prompt you for confirmation. Type `yes` to confirm.

**What’s happening here?**

- Terraform will delete all resources it previously created, such as the VPC, EC2 instances, RDS database, S3 bucket, CloudFront distribution, and more.

---

### **Resource Breakdown: What Each Section Does**

1. **VPC and Subnets**:
    - Creates a **VPC** for isolating AWS resources, with both public and private subnets to host different services.
    - Public subnets are for resources that need to communicate with the internet (like the ALB), and private subnets are for internal resources (like EC2 instances and RDS).
2. **Internet Gateway and NAT Gateway**:
    - **Internet Gateway** allows instances in public subnets to communicate with the internet.
    - **NAT Gateway** allows instances in private subnets to access the internet without exposing themselves publicly.
3. **Security Groups**:
    - Define rules for allowing or restricting network traffic to EC2, ALB, and RDS. For example, allowing HTTP traffic on port 80 to the web servers and allowing only internal access to the database.
4. **Application Load Balancer (ALB)**:
    - Distributes incoming traffic across multiple EC2 instances to ensure availability and scalability.
5. **EC2 Instances**:
    - EC2 instances run the backend application. They are hosted in private subnets and only accessible through the load balancer.
6. **Auto Scaling (Optional)**:
    - Configures the auto-scaling of EC2 instances based on traffic demand. This ensures that the application can scale dynamically to handle increased load and reduce costs when traffic is low.
7. **RDS MySQL Database**:
    - A highly available, multi-AZ **RDS MySQL** instance that serves as the backend database for the application.
8. **S3 Bucket and CloudFront**:
    - **S3** stores static content (like HTML, CSS, images) for the web application.
    - **CloudFront** is a CDN that accelerates the delivery of static assets by caching them at edge locations globally.

---

### **10. Best Practices**

- **Versioning**: Use version control for your Terraform files. This helps track infrastructure changes and collaborate more efficiently.
- **Remote State Management**: Store your Terraform state file remotely (e.g., using AWS S3 with state locking via DynamoDB) to ensure consistency across teams.
- **Cost Control**: Regularly review your AWS cost and usage reports to ensure resources are being used efficiently.
- **Security**: Rotate IAM credentials regularly, use IAM roles with the least privileges necessary, and enable multi-factor authentication (MFA) on your AWS account.

---

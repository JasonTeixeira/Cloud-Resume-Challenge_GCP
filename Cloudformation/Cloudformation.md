## Main Cloudformation File

### **Step-by-Step Instructional Breakdown for the CloudFormation Project**

This guide walks you through using AWS CloudFormation to build a highly available web application with AWS services, including CloudFront, RDS MySQL, EC2, S3, and related infrastructure. The CloudFormation template provisions the entire infrastructure stack and automates deployments.

---

### **1. Prerequisites**

Before we start, ensure the following:

- **AWS Account**: You need an AWS account with permissions to create resources like VPC, EC2, RDS, S3, CloudFront, etc.
- **AWS CLI Installed**: Install and configure AWS CLI:
    
    ```bash
    bash
    Copy code
    aws configure
    
    ```
    
    Enter your access key, secret key, and preferred region.
    
- **IAM Role/Permissions**: Ensure your user has the necessary IAM roles and policies to manage AWS resources (EC2, VPC, RDS, S3, CloudFront, etc.).

---

### **2. CloudFormation Template Setup**

Create a directory for your CloudFormation template, or navigate to where you'll store the file:

```bash
bash
Copy code
mkdir aws-web-app-cloudformation
cd aws-web-app-cloudformation

```

You can use the provided CloudFormation template to define the required resources.

---

### **3. Create a CloudFormation Stack**

To create the infrastructure, you’ll use AWS CloudFormation by either uploading the template file through the AWS Management Console or using the AWS CLI.

1. **Using AWS Console**:
    - Navigate to **CloudFormation** on the AWS Management Console.
    - Click on **Create Stack** and select **Upload a template file**.
    - Upload your CloudFormation template (`template.yaml` or `template.json`) and follow the wizard to configure stack parameters like VPC CIDR blocks, EC2 instance types, and database credentials.
2. **Using AWS CLI**:
If you want to deploy the stack using the AWS CLI, run the following command:
    
    ```bash
    bash
    Copy code
    aws cloudformation create-stack --stack-name MyWebAppStack --template-body file://template.yaml
    
    ```
    
    **What’s happening here?**
    
    - AWS CloudFormation processes your template and provisions the necessary resources based on the file’s instructions. This includes creating a VPC, EC2 instances, RDS databases, S3 bucket, and CloudFront distribution.

---

### **4. Review the CloudFormation Stack**

You can review the status of your stack using the AWS CLI:

```bash
bash
Copy code
aws cloudformation describe-stacks --stack-name MyWebAppStack

```

Or review the resources on the AWS Management Console under CloudFormation.

---

### **5. Key Resources Provisioned**

- **VPC and Subnets**: A new VPC is created with public and private subnets across two availability zones for high availability. Public subnets host the Application Load Balancer (ALB), and private subnets host EC2 instances and the RDS MySQL database.
- **Internet Gateway and NAT Gateway**: The Internet Gateway enables public subnets to communicate with the internet. The NAT Gateway allows instances in private subnets to reach the internet without being exposed to it.
- **Security Groups**: The configuration creates security groups for EC2 instances, the load balancer, and other services, ensuring only necessary ports (e.g., 80 for HTTP) are open.
- **Application Load Balancer (ALB)**: Distributes incoming traffic across EC2 instances in multiple Availability Zones to ensure availability and scalability.
- **Auto Scaling Group**: Automatically adjusts the number of EC2 instances to match traffic demand, scaling up when demand is high and scaling down when traffic is low.
- **RDS MySQL**: Creates a highly available RDS MySQL instance with multi-AZ failover, providing a backend relational database.
- **S3 Bucket and CloudFront Distribution**: S3 stores static website content, while CloudFront provides a global content delivery network (CDN) to accelerate website performance and reduce latency for users worldwide.

---

### **6. Check Outputs**

The CloudFormation stack will output critical information, including the ALB DNS name, S3 bucket name, CloudFront distribution URL, and RDS MySQL endpoint.

If you used the AWS CLI, you can retrieve these outputs using:

```bash
bash
Copy code
aws cloudformation describe-stacks --stack-name MyWebAppStack --query "Stacks[0].Outputs"

```

If you’re using the AWS Console, navigate to **CloudFormation > Stacks**, select your stack, and check the **Outputs** tab for details.

Outputs may include:

- **LoadBalancerDNS**: URL for the application hosted on the Application Load Balancer.
- **S3BucketName**: Name of the S3 bucket storing static content.
- **CloudFrontDistribution**: URL of the CloudFront distribution to serve static content.
- **DBEndpoint**: Endpoint for connecting to the MySQL database.

---

### **7. Access Your Application**

Now that your infrastructure is deployed, access your web application by visiting the Application Load Balancer’s DNS:

```bash
bash
Copy code
http://<LoadBalancerDNS>

```

To test CloudFront’s performance for static content delivery:

```bash
bash
Copy code
http://<CloudFrontDistribution>

```

---

### **8. Modify CloudFormation Template (Optional)**

If you need to modify the infrastructure (e.g., instance sizes, scaling policies, etc.), update the CloudFormation template, and then use **Update Stack** from the AWS Console or CLI to apply changes:

**AWS Console**:

- Select your stack in the CloudFormation console.
- Click **Update**, modify the template or parameters, and apply changes.

**AWS CLI**:

```bash
bash
Copy code
aws cloudformation update-stack --stack-name MyWebAppStack --template-body file://template.yaml

```

CloudFormation will only modify the necessary resources, ensuring minimal disruption.

---

### **9. Delete the CloudFormation Stack (Optional)**

When you no longer need the infrastructure, clean it up by deleting the CloudFormation stack to avoid incurring unnecessary costs.

**AWS Console**:

- Select your stack in the CloudFormation console and choose **Delete Stack**.

**AWS CLI**:

```bash
bash
Copy code
aws cloudformation delete-stack --stack-name MyWebAppStack

```

This will delete all resources provisioned by the stack, including the VPC, EC2 instances, RDS database, S3 bucket, CloudFront distribution, and more.

---

### **Resource Breakdown: What Each Section Does**

1. **VPC and Subnets**:
    - **VPC**: Creates an isolated virtual network to host the AWS resources.
    - **Subnets**: Public subnets are for resources that need internet access (e.g., ALB), and private subnets for backend services (e.g., EC2, RDS).
2. **Internet Gateway and NAT Gateway**:
    - **Internet Gateway**: Allows internet communication for resources in public subnets.
    - **NAT Gateway**: Enables instances in private subnets to reach the internet without exposing them.
3. **Security Groups**:
    - Controls inbound and outbound traffic to EC2 instances, ALB, RDS, and other services. For example, it allows HTTP traffic (port 80) and restricts access to private subnets.
4. **Application Load Balancer (ALB)**:
    - Distributes incoming web traffic across multiple EC2 instances to ensure high availability and scalability.
5. **EC2 Instances**:
    - Hosts the application backend and is placed in private subnets for security.
6. **Auto Scaling Group**:
    - Automatically adjusts the number of EC2 instances based on traffic demand.
7. **RDS MySQL**:
    - Creates a multi-AZ MySQL database that serves as the backend for the web application, providing high availability and failover support.
8. **S3 and CloudFront**:
    - S3 stores static assets (like HTML, CSS, JavaScript), and CloudFront caches this content globally, improving performance for users worldwide.

---

### **10. Best Practices**

- **Template Versioning**: Use version control (e.g., Git) for your CloudFormation templates to track changes and collaborate effectively.
- **Parameterization**: Use CloudFormation parameters to make your templates more flexible (e.g., allow variable instance types or regions).
- **Cost Monitoring**: Regularly review AWS cost reports to ensure resources are being used efficiently and avoid unexpected charges.
- **Security**: Use least-privileged IAM roles, rotate keys regularly, and enable multi-factor authentication (MFA) for all accounts.

---

### **Conclusion**

By following this guide, you’ve successfully built and deployed a highly available, scalable, and secure web application on AWS using CloudFormation. The automated infrastructure deployment helps ensure repeatability and consistency, while the use of AWS services like CloudFront, RDS, EC2, and S3 ensures global performance, security, and reliability.

CloudFormation streamlines the process of managing AWS resources, allowing you to update or delete the infrastructure as needed. This setup provides a strong foundation for cloud-native applications that need to be both performant and scalable.

```yaml
yaml
Copy code
AWSTemplateFormatVersion: '2010-09-09'
Description: High-Availability Web Application on AWS with VPC, EC2, RDS, S3, CloudFront, and Auto Scaling

Parameters:
  InstanceType:
    Description: EC2 instance type
    Type: String
    Default: t3.micro
  DBUsername:
    Description: RDS MySQL admin username
    Type: String
    Default: admin
  DBPassword:
    Description: RDS MySQL admin password
    Type: String
    NoEcho: true
  AMIID:
    Description: Amazon Linux 2 AMI ID
    Type: String
    Default: ami-0c55b159cbfafe1f0
  VPCCIDR:
    Description: The CIDR block for the VPC
    Type: String
    Default: 10.0.0.0/16

Resources:
  # VPC
  VPC:
    Type: AWS::EC2::VPC
    Properties:
      CidrBlock: !Ref VPCCIDR
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: Name
          Value: high-availability-vpc

  # Internet Gateway
  InternetGateway:
    Type: AWS::EC2::InternetGateway
    Properties:
      Tags:
        - Key: Name
          Value: main-igw

  AttachGateway:
    Type: AWS::EC2::VPCGatewayAttachment
    Properties:
      VpcId: !Ref VPC
      InternetGatewayId: !Ref InternetGateway

  # Subnets
  PublicSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.1.0/24
      AvailabilityZone: !Select [0, !GetAZs ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: public-subnet-1

  PublicSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.2.0/24
      AvailabilityZone: !Select [1, !GetAZs ]
      MapPublicIpOnLaunch: true
      Tags:
        - Key: Name
          Value: public-subnet-2

  PrivateSubnet1:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.3.0/24
      AvailabilityZone: !Select [0, !GetAZs ]
      Tags:
        - Key: Name
          Value: private-subnet-1

  PrivateSubnet2:
    Type: AWS::EC2::Subnet
    Properties:
      VpcId: !Ref VPC
      CidrBlock: 10.0.4.0/24
      AvailabilityZone: !Select [1, !GetAZs ]
      Tags:
        - Key: Name
          Value: private-subnet-2

  # NAT Gateway and Elastic IP
  NatEIP:
    Type: AWS::EC2::EIP
    Properties:
      Domain: vpc

  NatGateway:
    Type: AWS::EC2::NatGateway
    Properties:
      AllocationId: !GetAtt NatEIP.AllocationId
      SubnetId: !Ref PublicSubnet1

  # Route Tables
  PublicRouteTable:
    Type: AWS::EC2::RouteTable
    Properties:
      VpcId: !Ref VPC
      Tags:
        - Key: Name
          Value: public-rt

  PublicRoute:
    Type: AWS::EC2::Route
    Properties:
      RouteTableId: !Ref PublicRouteTable
      DestinationCidrBlock: 0.0.0.0/0
      GatewayId: !Ref InternetGateway

  PublicSubnetRouteTableAssociation1:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet1
      RouteTableId: !Ref PublicRouteTable

  PublicSubnetRouteTableAssociation2:
    Type: AWS::EC2::SubnetRouteTableAssociation
    Properties:
      SubnetId: !Ref PublicSubnet2
      RouteTableId: !Ref PublicRouteTable

  # Security Groups
  ALBSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !Ref VPC
      GroupDescription: Allow inbound HTTP traffic to ALB
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0

  EC2SecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      VpcId: !Ref VPC
      GroupDescription: Allow HTTP traffic to EC2 from ALB
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          SourceSecurityGroupId: !Ref ALBSecurityGroup

  # Load Balancer (ALB)
  LoadBalancer:
    Type: AWS::ElasticLoadBalancingV2::LoadBalancer
    Properties:
      Name: app-lb
      Scheme: internet-facing
      SecurityGroups: [!Ref ALBSecurityGroup]
      Subnets: [!Ref PublicSubnet1, !Ref PublicSubnet2]

  TargetGroup:
    Type: AWS::ElasticLoadBalancingV2::TargetGroup
    Properties:
      VpcId: !Ref VPC
      Protocol: HTTP
      Port: 80
      TargetType: instance
      Name: app-tg

  LoadBalancerListener:
    Type: AWS::ElasticLoadBalancingV2::Listener
    Properties:
      LoadBalancerArn: !Ref LoadBalancer
      Port: 80
      Protocol: HTTP
      DefaultActions:
        - Type: forward
          TargetGroupArn: !Ref TargetGroup

  # EC2 Instances
  WebInstance1:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AMIID
      InstanceType: !Ref InstanceType
      SubnetId: !Ref PrivateSubnet1
      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      Tags:
        - Key: Name
          Value: WebServer-1

  WebInstance2:
    Type: AWS::EC2::Instance
    Properties:
      ImageId: !Ref AMIID
      InstanceType: !Ref InstanceType
      SubnetId: !Ref PrivateSubnet2
      SecurityGroupIds:
        - !Ref EC2SecurityGroup
      Tags:
        - Key: Name
          Value: WebServer-2

  # RDS MySQL Database
  DBSubnetGroup:
    Type: AWS::RDS::DBSubnetGroup
    Properties:
      DBSubnetGroupDescription: Subnet group for RDS MySQL
      SubnetIds:
        - !Ref PrivateSubnet1
        - !Ref PrivateSubnet2
      DBSubnetGroupName: my-db-subnet-group

  RDSMySQL:
    Type: AWS::RDS::DBInstance
    Properties:
      AllocatedStorage: 20
      DBInstanceClass: db.t3.micro
      Engine: mysql
      MasterUsername: !Ref DBUsername
      MasterUserPassword: !Ref DBPassword
      MultiAZ: true
      DBSubnetGroupName: !Ref DBSubnetGroup
      PubliclyAccessible: false

  # S3 Bucket for Static Content
  S3Bucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: high-availability-webapp-bucket
      AccessControl: PublicRead
      VersioningConfiguration:
        Status: Enabled
      Tags:
        - Key: Name
          Value: StaticContentBucket

  # CloudFront Distribution
  CloudFrontOriginAccessIdentity:
    Type: AWS::CloudFront::CloudFrontOriginAccessIdentity
    Properties:
      CloudFrontOriginAccessIdentityConfig:
        Comment: Origin access identity for S3 bucket

  CloudFrontDistribution:
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: true
        DefaultRootObject: index.html
        Origins:
          - DomainName: !GetAtt S3Bucket.RegionalDomainName
            Id: S3-static-content
            S3OriginConfig:
              OriginAccessIdentity: !Sub "origin-access-identity/cloudfront/${CloudFrontOriginAccessIdentity}"
        DefaultCacheBehavior:
          TargetOriginId: S3-static-content
          ViewerProtocolPolicy: redirect-to-https
          AllowedMethods: [GET, HEAD]
          CachedMethods: [GET, HEAD]
          ForwardedValues:
            QueryString: false
            Cookies:
              Forward: none
        ViewerCertificate:
          CloudFrontDefaultCertificate: true

Outputs:
  LoadBalancerDNS:
    Description: DNS name of the load balancer
    Value: !GetAtt LoadBalancer.DNSName
  S3BucketName:
    Description: Name of the S3 bucket
    Value: !Ref S3Bucket
  CloudFrontDistributionURL:
    Description: CloudFront Distribution URL
    Value: !GetAtt CloudFrontDistribution.DomainName
  RDSMySQLEndpoint:
    Description: Endpoint for the MySQL RDS instance
    Value: !GetAtt RDSMySQL.Endpoint.Address

```

### Breakdown of the CloudFormation Template:

1. **VPC**: A VPC with public and private subnets is created across two availability zones.
2. **Internet Gateway & NAT Gateway**: The Internet Gateway is attached for public subnets, and a NAT Gateway is created for private subnets.
3. **Security Groups**: Security groups are created for the ALB and EC2 instances.
4. **Application Load Balancer**: The ALB is created and associated with target groups for the EC2 instances.
5. **EC2 Instances**: Two EC2 instances are provisioned within private subnets.
6. **RDS MySQL**: A MySQL database is deployed with multi-AZ failover support.
7. **S3 and CloudFront**: S3 is used for static content, and CloudFront provides a global CDN for delivering content with low latency.
8. **Outputs**: Key information like the Load Balancer DNS, S3 bucket name, CloudFront distribution URL, and RDS endpoint is provided as outputs.

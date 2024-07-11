# Project Write-Up: Setting Up a 3-Tier Architecture with MySQL-PHP Based App on AWS  

## Overview  
This project demonstrates the setup of a 3-tier architecture for a MySQL-PHP based application on AWS. The architecture consists of three availability zones (AZs): 1a, 1b, and 1c. Each AZ contains three subnets: one for the web tier (public), one for the app tier (private), and one for the database (DB) tier with RDS (private). A load balancer distributes traffic between the web instances and app instances.

The VPC range for this setup is 172.20.0.0/20.

- Architecture Diagram
```plaintext
             Internet
                 |
          +---------------+
          |   ALB (Web)   |
          +---------------+
                 |
          +---------------+
          |   Web Tier    |
          +---------------+
           |     |     |
      AZ 1a |  AZ 1b | AZ 1c
      +-----+ +-----+ +-----+
      | Web | | Web | | Web |
      +-----+ +-----+ +-----+
           |     |     |
          +---------------+
          |   ALB (App)   |
          +---------------+
                 |
          +---------------+
          |   App Tier    |
          +---------------+
           |     |     |
      AZ 1a |  AZ 1b | AZ 1c
      +-----+ +-----+ +-----+
      | App | | App | | App |
      +-----+ +-----+ +-----+
                 |
          +---------------+
          |    RDS DB     |
          +---------------+
           |     |     |
      AZ 1a |  AZ 1b | AZ 1c
      +-----+ +-----+ +-----+
      | DB  | | DB  | | DB  |
      +-----+ +-----+ +-----+
```
## Step-by-Step Setup Using AWS Management Console  

1. VPC Creation

- Navigate to the VPC Dashboard in the AWS Management Console.  
- Click on "Create VPC".  
- Enter the following details:  
  - Name tag: MyVPC  
  - IPv4 CIDR block: 172.20.0.0/20  
- Click "Create VPC".  


2. Subnets Creation  

- Web Tier Subnets (Public)

  - Navigate to "Subnets" in the VPC Dashboard.  
  - Click on "Create subnet".  
  - Enter the following details for each AZ:  
    - Name tag: WebSubnet-AZ1a  
    - VPC: MyVPC  
    - Availability Zone: us-east-1a  
    - IPv4 CIDR block: 172.20.1.0/24  
  - Repeat the process for us-east-1b with 172.20.2.0/24 and us-east-1c with 172.20.3.0/24.  

- App Tier Subnets (Private)

  - Repeat the above steps with:  
    - Name tags: AppSubnet-AZ1a, AppSubnet-AZ1b, AppSubnet-AZ1c  
    - IPv4 CIDR blocks: 172.20.4.0/24, 172.20.5.0/24, 172.20.6.0/24  
    - Availability Zones: us-east-1a, us-east-1b, us-east-1c  

- DB Tier Subnets (Private)  

  - Repeat the above steps with:  
    - Name tags: DBSubnet-AZ1a, DBSubnet-AZ1b, DBSubnet-AZ1c  
    - IPv4 CIDR blocks: 172.20.7.0/24, 172.20.8.0/24, 172.20.9.0/24  
    - Availability Zones: us-east-1a, us-east-1b, us-east-1c

3. Internet Gateway and Route Tables  

- Create Internet Gateway and Attach to VPC  

  - Navigate to "Internet Gateways" in the VPC Dashboard.  
    - Click on "Create internet gateway".  
    - Enter a name tag, e.g., MyIGW.  
    - Click "Create internet gateway".  
    - Select the created internet gateway, click on "Actions", and choose "Attach to VPC".  
    - Select MyVPC and click "Attach internet gateway".  

- Create Route Table for Public Subnets  
  
  - Navigate to "Route Tables" in the VPC Dashboard.  
  - Click on "Create route table".  
  - Enter a name tag, e.g., PublicRouteTable.  
  - VPC: MyVPC  
  - Click "Create route table".  
  - Select the created route table, click on "Actions", and choose "Edit routes".  
  - Click "Add route" and enter the following details:  
    - Destination: 0.0.0.0/0  
    - Target: Select the internet gateway (MyIGW)  
  - Click "Save routes".  

- Associate Route Table with Public Subnets  

  - Select the PublicRouteTable, click on the "Subnet associations" tab, and then click "Edit subnet associations".  
  - Select the subnets WebSubnet-AZ1a, WebSubnet-AZ1b, and WebSubnet-AZ1c.  
  - Click "Save".  

- Enable Public IP Assignment for Public Subnets  
 
  - Navigate to "Subnets" in the VPC Dashboard.  
  - Select each web subnet (WebSubnet-AZ1a, WebSubnet-AZ1b, WebSubnet-AZ1c), click on "Actions", and choose "Modify auto-assign IP settings".  
  - Check "Enable auto-assign public IPv4 address" and click "Save".  

4. NAT Gateway for Private Subnets  

- Create Elastic IP  
  
  - Navigate to "Elastic IPs" in the EC2 Dashboard.  
  - Click on "Allocate Elastic IP address".  
  - Click "Allocate".  

- Create NAT Gateway  

  - Navigate to "NAT Gateways" in the VPC Dashboard.  
  - Click on "Create NAT gateway".  
  - Enter the following details:  
    - Name: MyNATGateway  
    - Subnet: Select one of the web subnets (e.g., WebSubnet-AZ1a)  
    - Elastic IP allocation ID: Select the allocated Elastic IP  
  - Click "Create NAT gateway".  

- Create Route Table for Private Subnets  
  
- Navigate to "Route Tables" in the VPC Dashboard.  
- Click on "Create route table".  
- Enter a name tag, e.g., PrivateRouteTable.  
- VPC: MyVPC  
- Click "Create route table".  
- Select the created route table, click on "Actions", and choose "Edit routes".  
- Click "Add route" and enter the following details:  
  - Destination: 0.0.0.0/0  
  - Target: Select the NAT gateway (MyNATGateway)  
- Click "Save routes".  

- Associate Route Table with Private Subnets  
  
- Select the PrivateRouteTable, click on the "Subnet associations" tab, and then click "Edit subnet associations".  
- Select the subnets AppSubnet-AZ1a, AppSubnet-AZ1b, AppSubnet-AZ1c, DBSubnet-AZ1a, DBSubnet-AZ1b, and DBSubnet-AZ1c.  
- Click "Save".  

5. Deploy EC2 Instances for Web and App Tiers  
  
- Web Tier Instances  

- Navigate to "Instances" in the EC2 Dashboard.  
- Click on "Launch instances".  
- Enter the following details for each instance:  
  - Name: WebInstance-AZ1a, WebInstance-AZ1b, WebInstance-AZ1c  
  - AMI: Select an appropriate AMI (e.g., Amazon Linux 2)  
  - Instance type: t2.micro  
  - Key pair: Select your key pair  
  - Network settings:  
    - VPC: MyVPC  
    - Subnet: Select the respective web subnet (WebSubnet-AZ1a, WebSubnet-AZ1b, WebSubnet-AZ1c)  
    - Auto-assign public IP: Enabled  
  - Security group: Create a new security group allowing HTTP (port 80) and SSH (port 22) access  
- Click "Launch instance".

- App Tier Instances  
////////////////////////////////////
Repeat the above steps with:
Name: AppInstance-AZ1a, AppInstance-AZ1b, AppInstance-AZ1c
Subnet: Select the respective app subnet (AppSubnet-AZ1a, AppSubnet-AZ1b, AppSubnet-AZ1c)
Auto-assign public IP: Disabled
Security group: Create a new security group allowing access from the web tier instances on the necessary ports (e.g., HTTP port 80, SSH port 22)
Setup RDS Instance

Navigate to "Databases" in the RDS Dashboard.
Click on "Create database".
Select "Standard Create".
Choose "MySQL" as the engine type.
Select the "Free tier" template.
Enter the following details:
DB instance identifier: MyDBInstance
Master username: admin
Master password: yourpassword
Click "Next".
In the "Connectivity" section, select the following:
VPC: MyVPC
Subnet group: Create a new subnet group including the DB subnets (DBSubnet-AZ1a, DBSubnet-AZ1b, DBSubnet-AZ1c)
Public access: No
Security group: Create a new security group allowing access from the app tier instances
Click "Create database".
Setup Load Balancers

Web Load Balancer

Navigate to "Load Balancers" in the EC2 Dashboard.
Click on "Create Load Balancer" and select "Application Load Balancer".
Enter the following details:
Name: WebALB
Scheme: Internet-facing
IP address type: IPv4
Click "Next: Configure Security Settings".
In "Configure Security Groups", select the security group allowing HTTP access.
Click "Next: Configure Routing".
Create a new target group for the web tier instances.
Click "Next: Register Targets".
Register the web tier instances.
Click "Create".
App Load Balancer

Repeat the above steps with:
Name: AppALB
Scheme: Internal
Target group: Create a new target group for the app tier instances
Register the app tier instances
Configure Security Groups and IAM Roles

Ensure security groups allow necessary traffic between tiers.
Create IAM roles for EC2 instances to allow access to S3, CloudWatch, etc.
Deploy the MySQL-PHP Application

SSH into the web tier instances.
Install Apache, PHP, and other necessary packages.
Deploy the PHP application code from your Git repository.
Configure the application to connect to the RDS database.
Testing and Verification

Access the application through the web load balancer's DNS name.
Verify the application functionality.
Ensure proper communication between web, app, and DB tiers.
By following these steps, you can set up a robust 3-tier architecture for a MySQL-PHP based application on AWS, leveraging the AWS Management Console for configuration and deployment.







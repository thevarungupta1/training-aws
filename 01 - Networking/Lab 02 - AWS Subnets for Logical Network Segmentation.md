# Lab 02: AWS Subnets for Logical Network Segmentation

## Objective
Learn how to create a logically segmented network within a single VPC by designing and configuring multiple subnets. This lab will cover:
- Creating a VPC
- Creating both public and private subnets
- Setting up an Internet Gateway for the public subnet
- Configuring a NAT Gateway for the private subnet
- Launching EC2 instances to test connectivity

## Prerequisites
- **AWS Account:** An active AWS account with permissions to create VPCs, subnets, gateways, route tables, and EC2 instances.
- **AWS Management Console or CLI:** Familiarity with navigating the AWS Console or using the AWS CLI.
- **Basic AWS Networking Concepts:** Understanding of VPCs, subnets, routing, and security groups.

## Architecture Overview

```yaml
        +----------------------- VPC -----------------------+
        |                                                  |
        |  +----------------+       +-----------------+    |
        |  |  Public Subnet |       |  Private Subnet |    |
        |  |  (10.0.1.0/24) |       |  (10.0.2.0/24)  |    |
        |  |                |       |                 |    |
        |  |   EC2-Public   |       |   EC2-Private   |    |
        |  +----------------+       +-----------------+    |
        |        |                           |             |
        |        |                           |             |
        |   Internet Gateway            NAT Gateway         |
        |        |                           |             |
        +----------------------------------------------------+
```

*Note:* The public subnet is configured to allow direct Internet access via an Internet Gateway, while the private subnet uses a NAT Gateway for outbound Internet access.

## Step-by-Step Instructions

### 1. Create a VPC
- **VPC Details:**
  - **Name:** LogicalNetworkVPC
  - **CIDR Block:** 10.0.0.0/16
- In the AWS Console, navigate to **VPC Dashboard > Your VPCs** and click **Create VPC**. Enter the details and create the VPC.

### 2. Create Subnets in the VPC

#### a. Create the Public Subnet
- **Subnet Details:**
  - **Name:** PublicSubnet
  - **CIDR Block:** 10.0.1.0/24
  - **VPC:** LogicalNetworkVPC
  - **Availability Zone:** Choose an AZ as per your preference.
- In the VPC Dashboard, select **Subnets > Create Subnet** and provide the above details.

#### b. Create the Private Subnet
- **Subnet Details:**
  - **Name:** PrivateSubnet
  - **CIDR Block:** 10.0.2.0/24
  - **VPC:** LogicalNetworkVPC
  - **Availability Zone:** Optionally choose the same or a different AZ.
- Create the subnet similarly in the VPC Dashboard.

### 3. Set Up the Internet Gateway

- **Create and Attach:**
  - In the VPC Dashboard, click on **Internet Gateways > Create Internet Gateway**.
  - **Name:** LogicalNetworkIGW
  - Once created, select the IGW and choose **Actions > Attach to VPC**, then attach it to LogicalNetworkVPC.

### 4. Configure the Public Subnet Route Table

- **Create or Modify Route Table:**
  - Identify or create a route table for the public subnet.
  - Add a route:
    - **Destination:** 0.0.0.0/0
    - **Target:** LogicalNetworkIGW
- Associate this route table with the PublicSubnet.

### 5. Set Up a NAT Gateway for the Private Subnet

- **Create a NAT Gateway:**
  - Go to **NAT Gateways** in the VPC Dashboard.
  - **Name:** LogicalNATGateway
  - **Subnet:** Select PublicSubnet (since the NAT Gateway requires a public IP).
  - **Elastic IP:** Allocate a new Elastic IP or use an existing one.
- Wait until the NAT Gateway is available.

### 6. Configure the Private Subnet Route Table

- **Create or Modify Route Table:**
  - Create a new route table for the private subnet.
  - Add a route:
    - **Destination:** 0.0.0.0/0
    - **Target:** LogicalNATGateway
- Associate this route table with the PrivateSubnet.

### 7. Launch EC2 Instances

#### a. EC2 Instance in the Public Subnet
- Launch an instance (e.g., Amazon Linux 2) in PublicSubnet.
- Ensure you assign a public IP so that it can be accessed directly.
- Adjust the security group to allow SSH (port 22) from your IP.

#### b. EC2 Instance in the Private Subnet
- Launch an instance in PrivateSubnet.
- Do **not** assign a public IP.
- Configure the security group to allow SSH (if needed for internal access) from the PublicSubnet or your management network.
- This instance will access the Internet (for software updates, etc.) via the NAT Gateway.

### 8. Validate the Setup

- **Public Instance:** Verify that the public EC2 instance can be reached from the Internet using its public IP.
- **Private Instance:** Log into the public instance and try to SSH into the private instance using its private IP, or use it to access the Internet (e.g., run `curl http://checkip.amazonaws.com/` to verify NAT Gateway functionality).

### 9. Clean Up Resources
After completing your tests, ensure you clean up to avoid incurring charges:
- Terminate both EC2 instances.
- Delete the NAT Gateway.
- Detach and delete the Internet Gateway.
- Delete the subnets and the VPC.

## Lab Discussion Points

- **Subnet Segmentation:** Understand how subnets logically partition a VPC and isolate resources.
- **Public vs. Private Subnets:** Learn the differences between subnets that are directly reachable from the Internet and those that are isolated.
- **Routing and Gateways:** Appreciate the role of route tables, Internet Gateways, and NAT Gateways in managing traffic flow.
- **Security Best Practices:** Recognize the importance of security groups and proper subnet design to minimize exposure and improve security.

This lab exercise provides hands-on experience with AWS networking fundamentals, teaching you how to create a
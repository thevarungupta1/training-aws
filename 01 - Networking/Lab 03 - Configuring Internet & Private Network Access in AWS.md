# Lab 03: Configuring Internet & Private Network Access in AWS

## Objective
In this lab, you'll configure a VPC with both public and private subnets to provide Internet access where needed. You will:
- Create a VPC with two subnets.
- Enable Internet access in the public subnet via an Internet Gateway.
- Enable outbound Internet access in the private subnet via a NAT Gateway.
- Launch EC2 instances in each subnet to test connectivity.

## Prerequisites
- **AWS Account:** Active AWS account with permissions to create VPCs, subnets, gateways, route tables, and EC2 instances.
- **Basic Networking Knowledge:** Familiarity with VPC concepts, subnets, routing, and security groups.
- **AWS Management Console or CLI:** Comfort with navigating the AWS Console or using CLI commands.

## Architecture Overview

```yaml
        +----------------------- VPC (10.0.0.0/16) -----------------------+
        |                                                                |
        |   +------------------+           +-------------------------+ |
        |   |   Public Subnet  |           |    Private Subnet       | |
        |   |   (10.0.1.0/24)  |           |    (10.0.2.0/24)        | |
        |   |  EC2-Public      |           |  EC2-Private            | |
        |   +--------|---------+           +-----------|-------------+ |
        |            |                                   |               |
        |   Internet Gateway                          NAT Gateway         |
        +---------------------------------------------------------------+
```

*Note:* The public subnet is directly connected to the Internet via an Internet Gateway, while the private subnet uses a NAT Gateway to access the Internet for updates or outbound connections without being directly exposed.

## Step-by-Step Instructions

### 1. Create a VPC
- **VPC Details:**
  - **Name:** InternetPrivateVPC
  - **CIDR Block:** 10.0.0.0/16
- In the AWS Console, navigate to **VPC Dashboard > Your VPCs** and click **Create VPC**. Enter the details and create the VPC.

### 2. Create Subnets

#### a. Create the Public Subnet
- **Subnet Details:**
  - **Name:** PublicSubnet
  - **CIDR Block:** 10.0.1.0/24
  - **VPC:** InternetPrivateVPC
  - **Availability Zone:** Choose one based on your preference.
- In the VPC Dashboard, select **Subnets > Create Subnet** and provide the details.

#### b. Create the Private Subnet
- **Subnet Details:**
  - **Name:** PrivateSubnet
  - **CIDR Block:** 10.0.2.0/24
  - **VPC:** InternetPrivateVPC
  - **Availability Zone:** You can choose the same or a different Availability Zone.
- Create this subnet using the VPC Dashboard.

### 3. Set Up the Internet Gateway
- **Create an Internet Gateway:**
  - Navigate to **VPC Dashboard > Internet Gateways** and click **Create Internet Gateway**.
  - **Name:** InternetPrivateIGW
- **Attach the IGW to the VPC:**
  - Select the newly created IGW, choose **Actions > Attach to VPC**, and attach it to InternetPrivateVPC.

### 4. Configure the Public Subnet Route Table
- **Route Table Configuration:**
  - Find or create a route table for the PublicSubnet.
  - Add a route:
    - **Destination:** 0.0.0.0/0
    - **Target:** InternetPrivateIGW
- Associate this route table with PublicSubnet.

### 5. Set Up a NAT Gateway for Private Subnet
- **Create a NAT Gateway:**
  - Go to **VPC Dashboard > NAT Gateways** and click **Create NAT Gateway**.
  - **Name:** PrivateNATGateway
  - **Subnet:** Choose PublicSubnet (NAT Gateway must reside in a public subnet).
  - **Elastic IP:** Allocate a new Elastic IP address or use an existing one.
- Wait until the NAT Gateway status is available.

### 6. Configure the Private Subnet Route Table
- **Route Table Configuration:**
  - Create or select a route table dedicated for the PrivateSubnet.
  - Add a route:
    - **Destination:** 0.0.0.0/0
    - **Target:** PrivateNATGateway
- Associate this route table with PrivateSubnet.

### 7. Launch EC2 Instances

#### a. EC2 Instance in the Public Subnet
- Launch an EC2 instance (e.g., Amazon Linux 2) in PublicSubnet.
- Enable auto-assign Public IP so the instance is reachable from the Internet.
- Configure the security group to allow inbound SSH (port 22) from your IP.

#### b. EC2 Instance in the Private Subnet
- Launch another EC2 instance in PrivateSubnet.
- Do not assign a public IP.
- Configure its security group to allow SSH access from the PublicSubnet or a designated management IP.
- This instance will use the NAT Gateway for outbound Internet connectivity (e.g., for software updates).

### 8. Validate the Setup

- **Public Instance Validation:**
  - Access the public instance via its public IP (using SSH) to confirm it is reachable from the Internet.
- **Private Instance Validation:**
  - From the public instance, SSH into the private instance using its private IP address.
  - Alternatively, log into the private instance via a bastion host or Session Manager and run a command like `curl http://checkip.amazonaws.com/` to verify outbound Internet access via the NAT Gateway.

### 9. Clean Up Resources
After testing, clean up to avoid ongoing charges:
- Terminate both EC2 instances.
- Delete the NAT Gateway.
- Detach and delete the Internet Gateway.
- Delete the subnets and the VPC.
- Remove any associated route tables if not automatically cleaned up.

## Lab Discussion Points

- **Internet vs. Private Access:** Understand the differences in accessing resources directly from the Internet versus accessing them securely through a NAT Gateway.
- **Security Considerations:** Learn how public instances can be exposed to the Internet while keeping private instances shielded.
- **Routing Concepts:** Examine how route tables and gateways (Internet and NAT) direct traffic appropriately within your VPC.
- **Cost Considerations:** Recognize that while NAT Gateways offer secure outbound access, they may incur additional costs based on data transfer.

This lab exercise offers hands-on experience in setting up an AWS network that separates public and p
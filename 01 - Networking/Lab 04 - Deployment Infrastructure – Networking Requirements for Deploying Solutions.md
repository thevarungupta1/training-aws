# Lab 04: Deployment Infrastructure – Networking Requirements for Deploying Solutions

## Objective
In this lab, you will design and deploy a network infrastructure that supports a multi-tier application solution on AWS. You will create a VPC, configure public and private subnets, set up necessary gateways, and define security controls to meet the networking requirements for deploying your solution.

## Prerequisites
- **AWS Account:** Active AWS account with permissions to create VPCs, subnets, EC2 instances, and networking components.
- **Networking Fundamentals:** Familiarity with VPC concepts, subnets, routing, security groups, NAT Gateways, and Internet Gateways.
- **Application Architecture Knowledge:** Basic understanding of multi-tier applications (e.g., web, application, and database layers).

## Architecture Overview
This lab creates an environment with:
- A dedicated VPC with a CIDR block of 10.0.0.0/16.
- **Public Subnets:** Two subnets to host load balancers or other resources requiring direct Internet exposure.
- **Private Subnets:** Two subnets to host application and database servers that do not need direct Internet access.
- **Gateways:**
  - An **Internet Gateway** for public subnet connectivity.
  - A **NAT Gateway** to provide outbound Internet access for private subnets.
- **Security Groups:** Configured to allow only necessary communications between tiers.

**Network Diagram:**

```yaml
        +-------------------------- VPC (10.0.0.0/16) -------------------------+
        |                                                                     |
        |   +------------------+        +-------------------+                 |
        |   | Public Subnet 1  |        | Public Subnet 2   |                 |
        |   | (10.0.1.0/24)    |        | (10.0.2.0/24)     |                 |
        |   | [Load Balancer]  |        | [Load Balancer]   |                 |
        |   +--------|---------+       +---------|---------+                 |
        |            |                           |                           |
        |   ------------------- Internet Gateway (IGW) ----------------------|
        |                                                                     |
        |   +------------------+        +-------------------+                 |
        |   | Private Subnet 1 |        | Private Subnet 2  |                 |
        |   | (10.0.3.0/24)    |        | (10.0.4.0/24)     |                 |
        |   | [App Servers]    |        | [DB Servers]      |                 |
        |   +--------|---------+        +---------|---------+                 |
        |            |                           |                           |
        |         NAT Gateway (in a public subnet)                            |
        +---------------------------------------------------------------------+
```

## Step-by-Step Instructions

### 1. Create a VPC
- **VPC Details:**
  - **Name:** DeploymentVPC
  - **CIDR Block:** 10.0.0.0/16
- In the AWS Console, navigate to **VPC Dashboard > Your VPCs** and click **Create VPC**. Enter the details and create the VPC.

### 2. Create Subnets

#### a. Create Public Subnets
- **Public Subnet 1:**
  - **Name:** PublicSubnet1
  - **CIDR Block:** 10.0.1.0/24
  - **VPC:** DeploymentVPC
  - **Availability Zone:** Choose an AZ.
- **Public Subnet 2:**
  - **Name:** PublicSubnet2
  - **CIDR Block:** 10.0.2.0/24
  - **VPC:** DeploymentVPC
  - **Availability Zone:** Optionally select a different AZ for high availability.

#### b. Create Private Subnets
- **Private Subnet 1:**
  - **Name:** PrivateSubnet1
  - **CIDR Block:** 10.0.3.0/24
  - **VPC:** DeploymentVPC
  - **Availability Zone:** Choose an AZ.
- **Private Subnet 2:**
  - **Name:** PrivateSubnet2
  - **CIDR Block:** 10.0.4.0/24
  - **VPC:** DeploymentVPC
  - **Availability Zone:** Optionally select a different AZ.

### 3. Set Up an Internet Gateway
- **Create an Internet Gateway:**
  - Navigate to **VPC Dashboard > Internet Gateways > Create Internet Gateway**.
  - **Name:** DeploymentIGW
- **Attach the IGW to DeploymentVPC:**
  - Select the created IGW, click **Actions > Attach to VPC**, and attach it to DeploymentVPC.

### 4. Configure Route Tables for Public Subnets
- **Public Route Table:**
  - Create or select a route table for the public subnets.
  - Add a route with:
    - **Destination:** 0.0.0.0/0
    - **Target:** DeploymentIGW
- Associate this route table with both PublicSubnet1 and PublicSubnet2.

### 5. Set Up a NAT Gateway for Private Subnets
- **Create a NAT Gateway:**
  - Navigate to **VPC Dashboard > NAT Gateways > Create NAT Gateway**.
  - **Name:** DeploymentNAT
  - **Subnet:** Choose one of the public subnets (e.g., PublicSubnet1).
  - **Elastic IP:** Allocate a new Elastic IP address.
- Wait until the NAT Gateway becomes available.

### 6. Configure Route Tables for Private Subnets
- **Private Route Table:**
  - Create or select a route table for the private subnets.
  - Add a route:
    - **Destination:** 0.0.0.0/0
    - **Target:** DeploymentNAT
- Associate this route table with PrivateSubnet1 and PrivateSubnet2.

### 7. Configure Security Groups
- **Load Balancer Security Group:**
  - Allow inbound traffic on ports 80 and 443 (HTTP/HTTPS) from the Internet.
  - Allow health check traffic from within the VPC.
- **Application Server Security Group:**
  - Allow inbound traffic on required application ports (e.g., port 8080) from the Load Balancer Security Group.
  - Allow SSH (port 22) from your IP for management.
- **Database Server Security Group:**
  - Allow inbound traffic on database ports (e.g., MySQL: 3306, PostgreSQL: 5432) only from the Application Server Security Group.

### 8. Deploy Test Resources
- **Load Balancer:**
  - Deploy an Elastic Load Balancer in the public subnets. Configure it to forward traffic to a target group containing your application servers.
- **Application Servers:**
  - Launch EC2 instances in PrivateSubnet1. Install your application and register these instances with the load balancer.
- **Database Servers:**
  - Launch EC2 instances or provision an RDS instance in PrivateSubnet2.
- **Testing Connectivity:**
  - Access the load balancer's DNS from a browser or using `curl` to verify that traffic reaches the application servers.
  - Confirm that the application servers can connect to the database servers using private IP addresses.

### 9. Validate and Troubleshoot
- **Connectivity Checks:**
  - Verify that public subnets can reach the Internet via the IGW.
  - Ensure that private subnets can access the Internet for outbound requests via the NAT Gateway.
  - Use tools like `ping`, `traceroute`, or `curl` on EC2 instances to diagnose connectivity.
- **Security Controls:**
  - Ensure that security groups enforce the intended communication restrictions between tiers.

### 10. Clean Up Resources
After testing, clean up to avoid incurring charges:
- Terminate all EC2 and RDS instances.
- Delete the NAT Gateway.
- Detach and delete the Internet Gateway.
- Delete the subnets and the VPC.
- Remove any custom security groups and route tables if not automatically cleaned up.

## Lab Discussion Points
- **Network Segmentation:** The importance of segregating public and private resources to enhance security.
- **Routing Mechanisms:** How Internet and NAT Gateways facilitate traffic management.
- **Security Best Practices:** Implementing security groups to control access between different layers.
- **High Availability:** Deploying resources across multiple Availability Zones to improve resilience.
- **Cost Considerations:** Evaluating the cost implications of NAT Gateways, load balancers, and other networking resources.

This lab exercise demonstrates how to build a robust and secure AWS networking infrastructure that m
# Lab 01: AWS VPC Peering for Isolated Network Segments

## Objective
Learn how to:
- Create two isolated VPCs with non-overlapping CIDR blocks.
- Launch EC2 instances in separate VPC subnets.
- Establish a VPC Peering connection between the VPCs.
- Configure route tables and security groups to allow controlled inter-VPC communication.
- Validate connectivity between the instances.

## Prerequisites
- **AWS Account:** Ensure you have an active AWS account with permissions to create VPCs, EC2 instances, and modify route tables/security groups.
- **Familiarity with AWS Concepts:** Basic knowledge of VPCs, subnets, routing, security groups, and EC2.
- **AWS Management Console or CLI:** You can use the AWS Console or CLI for these steps.

Architecture Overview
```yaml
Copy
   +------------------+          VPC Peering          +--------------------+
   |      VPC-A     | 10.0.0.0/16  <===========>  |      VPC-B       | 192.168.0.0/16
   |                |                              |                  |
   |  Public Subnet | 10.0.1.0/24                  |  Public Subnet   | 192.168.1.0/24
   |   (EC2-A)      |                              |     (EC2-B)      |
   +----------------+                              +------------------+

```

*Note:* The VPC peering connection enables routing between VPC-A and VPC-B while keeping the segments isolated from other networks.

## Step-by-Step Instructions

### 1. Create Two VPCs

- **VPC-A:**
  - **CIDR Block:** 10.0.0.0/16
  - Go to **VPC Dashboard > Your VPCs** and click **Create VPC**.
  - Name it “VPC-A.”

- **VPC-B:**
  - **CIDR Block:** 192.168.0.0/16
  - Similarly, create another VPC and name it “VPC-B.”

### 2. Create Subnets in Each VPC

- **For VPC-A:**
  - Create a subnet with CIDR **10.0.1.0/24**.
  - Ensure it is in the desired Availability Zone.
  
- **For VPC-B:**
  - Create a subnet with CIDR **192.168.1.0/24**.
  - Select an Availability Zone (can be the same or different).

*Tip:* These subnets can be configured as “public” if you plan to assign public IP addresses; however, for testing peering, private IPs are sufficient.

### 3. Launch EC2 Instances in Each Subnet

- **EC2 in VPC-A (EC2-A):**
  - Launch an instance in the subnet (10.0.1.0/24) of VPC-A.
  - Choose an AMI (for example, Amazon Linux 2) and a small instance type.
  - Make sure to assign a key pair for SSH access.
  
- **EC2 in VPC-B (EC2-B):**
  - Launch an instance in the subnet (192.168.1.0/24) of VPC-B.
  - Use similar instance settings as above.

*Note:* Ensure the instances’ security groups initially allow basic SSH (port 22) from your IP for setup.

### 4. Create a VPC Peering Connection

- In the AWS Management Console:
  - Navigate to **VPC Dashboard > Peering Connections**.
  - Click **Create Peering Connection**.
    - **Requester VPC:** Select VPC-A.
    - **Accepter VPC:** Select VPC-B (if both VPCs are in the same account, the process is straightforward).
  - Provide a meaningful name tag, then create the connection.
- **Accept the Peering Request:**
  - In VPC-B, locate the pending peering connection and click **Accept Request**.

### 5. Update Route Tables

- **For VPC-A:**
  - Go to **Route Tables** associated with the subnet (10.0.1.0/24).
  - Edit routes to add:
    - **Destination:** 192.168.0.0/16
    - **Target:** The VPC peering connection ID.
    
- **For VPC-B:**
  - Locate the route table associated with the subnet (192.168.1.0/24).
  - Edit routes to add:
    - **Destination:** 10.0.0.0/16
    - **Target:** The same peering connection.

*Why this is needed:* The route tables must be explicitly told to forward traffic destined for the other VPC through the peering connection.

### 6. Adjust Security Groups

- **For EC2-A (in VPC-A):**
  - Edit the security group inbound rules to allow:
    - **Type:** SSH (TCP 22) and/or ICMP (for ping)
    - **Source:** 192.168.0.0/16 (VPC-B’s CIDR)
  
- **For EC2-B (in VPC-B):**
  - Similarly, allow inbound traffic:
    - **Type:** SSH (TCP 22) and/or ICMP
    - **Source:** 10.0.0.0/16 (VPC-A’s CIDR)

*Tip:* For a quick test, you may also temporarily allow all traffic between the two VPC CIDRs and later restrict to necessary ports.

### 7. Validate the VPC Peering Connection

- **Testing Connectivity:**
  - SSH into one of the EC2 instances (for example, EC2-A).
  - Use the private IP address of the other instance (EC2-B) to:
    - **Ping:** `ping <EC2-B-private-IP>`
    - **SSH (if desired):** `ssh ec2-user@<EC2-B-private-IP>`
  - Successful responses indicate that the peering connection and routing are correctly set up.

### 8. Clean Up Resources

After completing your tests, remember to:
- **Delete the VPC Peering Connection:** Navigate to the Peering Connections section and delete it.
- **Terminate the EC2 Instances:** In the EC2 Dashboard.
- **Delete the VPCs and Subnets:** In the VPC Dashboard, clean up the created VPCs to avoid ongoing charges.

## Lab Discussion Points

- **Routing Requirements:** Understanding that VPC peering requires manual route table updates for cross-VPC traffic.
- **Security Considerations:** Recognizing that security groups and network ACLs must allow desired traffic and that overly permissive rules can lead to security risks.
- **Peering Limitations:** Noting that VPC peering does not support transitive routing; if multiple VPCs need to communicate, consider using AWS Transit Gateway.
- **Cost Implications:** Remember that while VPC peering itself does not incur extra charges, data transfer between regions (if peered) may have associated costs.

# AWS Well-Architected Framework Lab Exercise

## Overview
In this lab, you will build a sample web application on AWS and review it against the AWS Well-Architected Framework. You will apply best practices from the five pillars—Operational Excellence, Security, Reliability, Performance Efficiency, and Cost Optimization—to create a robust, scalable, and secure architecture.

---

## AWS Well-Architected and the Six Pillars

**1. Operational Excellence Pillar**
The operational excellence pillar focuses on running and monitoring systems, and continually improving processes and procedures. Key topics include automating changes, responding to events, and defining standards to manage daily operations.

**2. Security Pillar**
The security pillar focuses on protecting information and systems. Key topics include confidentiality and integrity of data, managing user permissions, and establishing controls to detect security events.

**3. Reliability Pillar**
The reliability pillar focuses on workloads performing their intended functions and how to recover quickly from failure to meet demands. Key topics include distributed system design, recovery planning, and adapting to changing requirements.

**4. Performance Efficiency Pillar**
The performance efficiency pillar focuses on structured and streamlined allocation of IT and computing resources. Key topics include selecting resource types and sizes optimized for workload requirements, monitoring performance, and maintaining efficiency as business needs evolve.

**5. Cost Optimization Pillar**
The cost optimization pillar focuses on avoiding unnecessary costs. Key topics include understanding spending over time and controlling fund allocation, selecting resources of the right type and quantity, and scaling to meet business needs without overspending.

**6. Sustainability Pillar**
The sustainability pillar focuses on minimizing the environmental impacts of running cloud workloads. Key topics include a shared responsibility model for sustainability, understanding impact, and maximizing utilization to minimize required resources and reduce downstream impacts. 

----

## Lab Objectives
- **Understand the AWS Well-Architected Framework** by applying its five pillars to a real-world scenario.
- **Deploy a sample web applicatio**n using core AWS services.
- **Implement best practices** in areas such as security, monitoring, scalability, and cost management.
- **Analyze and improve** your architecture based on feedback from AWS tools and metrics.

----

## Prerequisites
- An active AWS account.
- Basic familiarity with AWS services such as EC2, VPC, IAM, Auto Scaling, ELB, CloudWatch, and S3.
- Basic understanding of networking concepts (e.g., subnets, security groups).
- Command-line experience for using the AWS CLI (optional but beneficial).

----

## Estimated Duration
Approximately 3–4 hours (depending on familiarity with AWS and the complexity of implementation).

-----

## Lab Environment Setup
### 1. AWS Account Setup:
    - Ensure you have an active AWS account.
    - Set up billing alerts to monitor your usage.
### 2. AWS CLI & SDKs:
    - (Optional) Install and configure the AWS CLI to run commands directly from your terminal.
### 3. Permissions:
    - Create an IAM user/role with administrative permissions for the duration of the lab (ensure to follow least privilege principles later).

-----

## Lab Tasks
### Task 1: Build the Foundation
### A. Create a Virtual Private Cloud (VPC)
- **Objective:** Isolate your resources within a secure network.
- **Steps:**
    1. Go to the VPC Dashboard.
    2. Create a new VPC with a CIDR block (e.g., 10.0.0.0/16).
    3. Create two subnets:
        - A public subnet (for load balancers and NAT gateways).
        - A private subnet (for your application servers).
    4. Configure an Internet Gateway and attach it to the VPC.
    5. Set up route tables for proper routing.

### B. Define Security Baselines
- **Objective:** Ensure security from the start.
- **Steps:**
    1. Create security groups for your EC2 instances:
        - Allow HTTP/HTTPS traffic from the load balancer.
        - Limit SSH access (e.g., allow from your IP only).
    2. Define IAM roles for EC2 instances to interact with other AWS services securely.
----

## Task 2: Deploy a Sample Web Application
### A. Launch an EC2 Instance (or use Elastic Beanstalk)
- **Objective:** Host your sample application.
- **Steps:**
    1. In the EC2 Dashboard, launch an instance in the public subnet.
    2. Select an Amazon Linux 2 or Ubuntu AMI.
    3. Attach the previously created IAM role.
    4. Install a simple web server (e.g., Apache or Nginx) and deploy a sample HTML page.
    5. Verify that the application is reachable via the public IP.

### B. Configure a Load Balancer & Auto Scaling Group
- **Objective:** Enhance reliability and performance.
- **Steps:**
    1. Set up an Application Load Balancer (ALB) and register your instance.
    2. Create an Auto Scaling Group (ASG) with a launch configuration/template:
        - Set minimum and maximum instance counts.
        - Attach the ASG to your ALB.
    3. Test the scaling by simulating load (e.g., using a load testing tool).

---- 

## Task 3: Integrate Operational Excellence
### A. Implement Monitoring & Logging
- **Objective:** Gain insights into application performance and operational health.
- **Steps:**
    1. Set up Amazon CloudWatch to monitor key metrics (CPU, memory, network).
    2. Create CloudWatch alarms to notify you of critical events (e.g., high CPU usage).
    3. Configure AWS CloudTrail to capture API activity.
    4. (Optional) Deploy a logging solution (like the ELK stack) for deeper analysis.

### B. Establish Automation and Runbooks
- **Objective:** Improve response time to issues.
- **Steps:**
    1. Use AWS Systems Manager to create and run automation documents (runbooks) for common tasks (e.g., instance recovery).
    2. Document procedures for backup, scaling events, and security incident response.

----- 
## Task 4: Enhance Security
### A. Strengthen Identity and Access Management (IAM)
- **Objective:** Ensure that the principle of least privilege is applied.
- **Steps:**
    1. Review your IAM policies and restrict permissions where possible.
    2. Enable Multi-Factor Authentication (MFA) for critical accounts.
    3. Regularly audit IAM roles and user permissions using AWS IAM Access Analyzer.

### B. Data Protection & Encryption
- **Objective:** Secure data in transit and at rest.
- **Steps:**
    1. Enable encryption on your EBS volumes and S3 buckets.
    2. Use AWS Key Management Service (KMS) to manage encryption keys.
    3. Validate SSL/TLS configuration on your load balancer.

------

## Task 5: Improve Reliability & Performance Efficiency
### A. Architect for Fault Tolerance
- **Objective:** Reduce downtime and service disruptions.
- **Steps:**
    1. Deploy resources across multiple Availability Zones (AZs).
    2. Set up health checks in your load balancer and auto scaling groups.
    3. Implement backup strategies using AWS Backup or manual snapshots.

### B. Optimize Performance
- **Objective:** Ensure efficient resource usage.
- **Steps:**
    1. Choose the appropriate EC2 instance types based on workload.
    2. Use Amazon CloudFront as a content delivery network (CDN) to reduce latency.
    3. Use AWS Elasticache to cache frequent queries or session data.

-----

## Task 6: Evaluate Cost Optimization
### A. Analyze Costs & Optimize Spending
- **Objective:** Maintain a cost-effective architecture.
- **Steps:**
    1. Use AWS Cost Explorer to review your spending.
    2. Identify underutilized resources and consider rightsizing.
    3. Set up AWS Budgets and alerts for proactive cost management.
    4. Evaluate options for Reserved Instances or Savings Plans if your usage is predictable.

----

## Post-Lab Reflection & Questions
After completing the hands-on tasks, take some time to answer the following questions:
1. **Operational Excellence:** How did the implemented monitoring and automation tools help in maintaining application health?
2. **Security:** What IAM practices and encryption strategies did you find most effective?
3. **Reliability:** How did deploying across multiple AZs improve the fault tolerance of your application?
4. **Performance Efficiency:** What optimizations did you apply to ensure the application responds quickly to user requests?
5. **Cost Optimization:** Which strategies would you adopt to further reduce costs without sacrificing performance?

-----

## Conclusion
This lab exercise not only walks you through the practical deployment of a web application but also reinforces how each pillar of the AWS Well-Architected Framework plays a vital role in building robust cloud architectures. By iterating on each task, you can continually refine your design to meet evolving business and technical requirements.
# Advanced Lab: Deploy a Multi-Container Application on AWS Fargate

## Overview
In this lab exercise, you will deploy a multi-container application on AWS Fargate with advanced capabilities including:
- **Blue/Green deployments** using AWS CodeDeploy
- **Auto Scaling** of your ECS Service based on CloudWatch metrics
- **Application Load Balancer (ALB)** configuration for dynamic traffic routing
- **Enhanced monitoring and logging** with AWS CloudWatch
- **Secrets and Parameter Management** integration (optional)

This lab is designed for users who already have a basic understanding of AWS Fargate and containerized applications.

## Prerequisites
- **AWS Account:** Active account with permissions to manage ECS, CodeDeploy, ALB, CloudWatch, IAM, and related resources.
- **IAM Permissions:** Ensure you have the necessary permissions to create ECS clusters, services, task definitions, ALB, CodeDeploy applications, and CloudWatch alarms.
- **AWS CLI (Optional):** For advanced configuration and automation tasks.
- **Docker Knowledge:** Familiarity with building and managing Docker containers.
- **Basic Networking:** Understanding of VPCs, subnets, and security groups.

---

## Step 1: Create an ECS Cluster with Fargate Capacity Provider

1. **Log in to the AWS Management Console.**
2. **Navigate to Amazon ECS:**  
   From the AWS Services menu, select **Amazon ECS**.
3. **Create a Cluster:**
   - Go to **Clusters** and click **Create Cluster**.
   - Choose **Networking only (Powered by AWS Fargate)**.
   - Enter a cluster name (e.g., `AdvancedFargateCluster`).
   - Select your VPC and subnets. For internet-facing tasks, ensure you select subnets with a NAT gateway or an internet gateway.
   - Click **Create**.

---

## Step 2: Create a Multi-Container Task Definition

1. **Navigate to Task Definitions:**
   - In the ECS console sidebar, click on **Task Definitions**.
   - Click **Create new Task Definition** and choose **Fargate**.
2. **Configure Task Definition:**
   - **Task Definition Name:** `AdvancedMultiContainerTask`
   - **Task execution role:** Select an IAM role that permits pulling container images and writing logs to CloudWatch. If needed, create a new one.
   - **Task Size:** Configure resources (e.g., Memory: 1024 MB, CPU: 0.5 vCPU).
3. **Add Container Definitions:**
   - **Frontend Container:**
     - **Container name:** `frontend`
     - **Image:** Use a public image (e.g., `nginx:latest`) or your custom built image.
     - **Port Mappings:** Map container port `80` to the host.
   - **Backend Container:**
     - **Container name:** `backend`
     - **Image:** Use a sample API image (e.g., a Node.js API such as `amazon/amazon-ecs-sample` or your own).
     - **Port Mappings:** Expose a port (e.g., container port `3000`).
   - **Environment Variables & Secrets (Optional):**
     - Configure sensitive data using AWS Systems Manager Parameter Store or Secrets Manager.
   - **Logging:** Under **Log configuration**, select **awslogs** and specify a log group (e.g., `/ecs/AdvancedFargateLab`).
4. **Create Task Definition:** Review the settings and click **Create**.

---

## Step 3: Configure an Application Load Balancer (ALB)

1. **Create an ALB:**
   - Navigate to the EC2 console, then **Load Balancers**, and click **Create Load Balancer**.
   - Choose **Application Load Balancer** and provide a name (e.g., `AdvancedFargateALB`).
   - Configure listeners (HTTP on port 80) and select at least two subnets.
2. **Create Target Groups:**
   - Create two target groups (one for the blue environment and one for green) using the **IP** target type.
   - Configure health checks (e.g., HTTP, path `/`).
3. **Security Groups:**  
   - Ensure the ALB security group permits inbound HTTP traffic from the internet.
   - Confirm that the ECS tasks’ security groups allow traffic from the ALB.

---

## Step 4: Create an ECS Service with Blue/Green Deployment

1. **Navigate to Your Cluster:**
   - In the ECS console, select `AdvancedFargateCluster`.
2. **Create Service:**
   - Click **Create Service**.
   - **Launch type:** Choose **Fargate**.
   - **Task Definition:** Select `AdvancedMultiContainerTask`.
   - **Service Name:** Enter a name (e.g., `AdvancedFargateService`).
   - **Number of Tasks:** Set initial desired count (e.g., 2).
3. **Configure Deployment Options:**
   - Enable **Blue/Green Deployment** by selecting the option to integrate with AWS CodeDeploy.
   - Choose the target groups created earlier for traffic shifting.
   - Configure deployment settings such as traffic shifting type (linear, canary, or all-at-once) and wait time.
4. **Networking:**
   - Select your VPC and subnets.
   - Enable **Auto-assign public IP** if the tasks are internet-facing.
5. **Load Balancing:**
   - Associate the ALB and specify the container port mappings (e.g., map `frontend` container port `80` to the ALB target group).
6. **Review and Create:**  
   - Verify all settings and click **Create Service**.
7. **Configure CodeDeploy:**
   - If prompted, configure the CodeDeploy application and deployment group with proper IAM roles and target group settings.

---

## Step 5: Enable Auto Scaling for the ECS Service

1. **Navigate to the ECS Service Auto Scaling:**
   - In the ECS console, select your service (`AdvancedFargateService`), then choose the **Auto Scaling** tab.
2. **Create Scaling Policies:**
   - **Scale Out Policy:**  
     Set a target CPU utilization threshold (e.g., 70%). Configure CloudWatch alarms to add tasks when CPU utilization exceeds the threshold.
   - **Scale In Policy:**  
     Similarly, configure a policy to reduce tasks when CPU utilization falls below a set threshold (e.g., 30%).
3. **Save Changes:**  
   - Confirm and apply your auto scaling settings.

---

## Step 6: Monitor and Verify the Deployment

1. **CloudWatch Logs & Metrics:**
   - Navigate to **CloudWatch Logs** to verify that container logs (from both frontend and backend) are streaming correctly.
   - Check **CloudWatch Metrics** for ECS service performance, including CPU and memory utilization.
2. **Access the Application:**
   - Retrieve the ALB DNS name from the EC2 console under **Load Balancers**.
   - Open a web browser and enter the ALB DNS name. Verify that the frontend (and by extension, the backend) is functioning as expected.
3. **Test Blue/Green Deployment:**
   - Trigger a new deployment by updating the task definition (e.g., changing an environment variable or container image tag).
   - Observe how CodeDeploy shifts traffic from the blue environment to the green environment gradually.

---

## Step 7: Clean Up Resources

1. **Stop and Delete the ECS Service:**
   - In the ECS console, navigate to your service and choose **Delete Service**.
2. **Delete the ECS Cluster:**
   - Once the service is removed, select your cluster (`AdvancedFargateCluster`) and click **Delete Cluster**.
3. **Remove ALB and Target Groups:**
   - Go to the EC2 console, select your ALB (`AdvancedFargateALB`) and target groups, and delete them.
4. **Clean Up CodeDeploy Applications (Optional):**
   - Delete the CodeDeploy application and deployment group if they are no longer needed.
5. **Remove CloudWatch Log Groups (Optional):**
   - Delete the CloudWatch log group `/ecs/AdvancedFargateLab` if you wish to free up resources.

---

## Lab Conclusion

In this advanced lab exercise, you:
- Deployed a multi-container application on AWS Fargate.
- Configured an Application Load Balancer for dynamic traffic routing.
- Set up blue/green deployments using AWS CodeDeploy for seamless updates.
- Enabled auto scaling for the ECS service based on CloudWatch metrics.
- Monitored the deployment using AWS CloudWatch for both logging and metrics.

This lab demonstrates how AWS Fargate can serve as a powerful serverless compute engine for containers, enabling sophisticated deployment patterns and operational practices without managing servers. Use these advanced techniques as a foundation to build production-grade, scalable containerized applications.

Happy Advanced Learning!

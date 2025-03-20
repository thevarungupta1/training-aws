# Lab: Deploy a Containerized Application on AWS Fargate

## Overview
In this lab exercise, you will:
- Create an Amazon ECS cluster configured for Fargate.
- Define a task that uses a sample container image.
- Launch your containerized application on Fargate.
- Validate the deployment by accessing the application.
- Clean up the resources to avoid unnecessary charges.

## Prerequisites
- **AWS Account:** Ensure you have an active AWS account with permissions to create ECS clusters, task definitions, and run Fargate tasks.
- **Basic Knowledge:** Familiarity with Docker concepts and container images.
- **Tools:** Access to the AWS Management Console (you can also use the AWS CLI for advanced tasks).

---

## Step 1: Create an ECS Cluster

1. **Log in to the AWS Management Console.**
2. **Navigate to Amazon ECS:**  
   From the AWS Services menu, select **Amazon ECS**.
3. **Create a Cluster:**
   - Click on **Clusters** in the left-hand navigation pane.
   - Click the **Create Cluster** button.
   - Choose the **Networking only (Powered by AWS Fargate)** option.
   - Provide a cluster name (e.g., `FargateLabCluster`).
   - For VPC and subnets, you can use the default settings or select your custom VPC/subnets.
   - Click **Create** to provision your cluster.

---

## Step 2: Create a Task Definition

1. **Navigate to Task Definitions:**
   - In the ECS console sidebar, click on **Task Definitions**.
   - Click **Create new Task Definition**.
2. **Select Launch Type:**
   - Choose **Fargate** as the launch type.
3. **Configure Task Definition:**
   - **Task Definition Name:** Enter a name (e.g., `FargateLabTask`).
   - **Task execution role:** Select an existing IAM role with permissions (or create a new one if prompted). This role allows Fargate to pull container images and log to CloudWatch.
   - **Task Size:** Set memory (e.g., 512 MB) and CPU (e.g., 0.25 vCPU) based on your needs.
4. **Add a Container Definition:**
   - Click **Add container**.
   - **Container name:** Provide a name (e.g., `my-app-container`).
   - **Image:** Use a public image such as `nginx:latest` or the AWS sample image `amazon/amazon-ecs-sample` if available.
   - **Port Mappings:** If using NGINX, map container port **80** to the host (this allows you to access the web server).
   - Optionally adjust other settings like environment variables or resource limits.
   - Click **Add**.
5. **Create Task Definition:**
   - Review your settings and click **Create**.

---

## Step 3: Run a Task on the Cluster

1. **Navigate to Your Cluster:**
   - From the ECS console, click on **Clusters** and select `FargateLabCluster`.
2. **Run New Task:**
   - Click the **Tasks** tab, then click **Run new Task**.
   - **Launch type:** Select **Fargate**.
   - **Task Definition:** Choose `FargateLabTask` (select the revision if applicable).
   - **Cluster:** Confirm it is your `FargateLabCluster`.
3. **Configure Networking:**
   - Choose your VPC and select at least one subnet.
   - **Auto-assign public IP:** Enable this option if you want your task accessible from the internet.
4. **Launch the Task:**
   - Click **Run Task**.
   - Wait a few minutes while AWS provisions your container on Fargate.

---

## Step 4: Verify the Deployment

1. **Check Task Status:**
   - In your cluster’s **Tasks** tab, ensure your task is in the `RUNNING` state.
2. **Retrieve the Public IP:**
   - Click on the running task to view details.
   - Under the **Networking** section, note the public IP address assigned.
3. **Access Your Application:**
   - Open a web browser and enter the public IP address.
   - If you used the `nginx:latest` image, you should see the default NGINX welcome page.

---

## Step 5: Clean Up Resources

1. **Stop the Running Task:**
   - In your cluster’s **Tasks** tab, select the running task and click **Stop**.
2. **Delete the ECS Cluster (Optional):**
   - If you no longer need the cluster, navigate to the **Clusters** section, select `FargateLabCluster`, and choose **Delete Cluster**.
3. **Remove Task Definitions (Optional):**
   - Under **Task Definitions**, select `FargateLabTask` and deregister any revisions you created.

---

## Lab Conclusion

In this lab exercise, you learned how to deploy a containerized application on AWS Fargate without managing servers. You created an ECS cluster, defined a Fargate task, deployed a container using a sample image, and validated your deployment by accessing the application through a public IP address. Finally, you practiced cleaning up your resources to prevent unwanted charges.

This hands-on lab introduces you to the basics of serverless container orchestration with AWS Fargate. For production use, consider exploring advanced topics such as auto-scaling, secure networking, logging with CloudWatch, and integration with CI/CD pipelines.

Happy Learning!

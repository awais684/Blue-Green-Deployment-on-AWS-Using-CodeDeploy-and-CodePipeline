# Blue-Green-Deployment-on-AWS-Using-CodeDeploy-and-CodePipeline

When you're learning AWS in depth, nothing teaches you better than building a real-world pipeline. In this project, I created a highly available and automated deployment architecture using EC2, CodeDeploy, Load Balancer, Auto Scaling Group, Launch Templates, GitHub, and CodePipeline.

This blog walks you through the exact process — every IAM role, installation, configuration, and deployment — so you can rebuild it on your own or use it as a learning reference.

## This setup ensures:

High availability
Automated deployments
Auto-healing servers
CI/CD with zero manual intervention

## 1️⃣ Creating IAM Roles

"IAM Role for EC2"

This EC2 role needs permissions for:

Administrator Access
AmazonS3FullAccess
AWSCodeDeployFullAccess
AWSCodePipelineFullAccess
AutoScaling Full Access
ELB Full Access

Create IAM Role → AWS Service → EC2 → Attach Policies above → Create Role
Attach this role later when launching the EC2 instance.

"IAM Role for CodeDeploy"

CodeDeploy uses this role to interact with your EC2 instances.

AdministratorAccess
AmazonS3FullAccess
Service: CodeDeploy

## 2️⃣ Launching an EC2 Instance (Amazon Linux)

- AMI: Amazon Linux 2
- Instance Type: t2.micro (Free tier)
- IAM Role: attach the EC2 role you created
- Security Group: open HTTP (80) + SSH (22)

SSH into the instance:

```
sudo yum install httpd -y
sudo systemctl start httpd
sudo systemctl enable httpd

```

## 3️⃣ Installing the CodeDeploy Agent

```
sudo yum install ruby -y
sudo yum install wget -y
cd /home/ec2-user
wget https://aws-codedeploy-us-east-1.s3.us-east-1.amazonaws.com/latest/install
chmod +x ./install
sudo ./install auto
sudo systemctl start codedeploy-agent
sudo systemctl enable codedeploy-agent

```
Verify:

```
sudo systemctl status codedeploy-agent

```

## 4️⃣ Creating an AMI from the EC2 Instance

Now that the EC2 instance has a proper role attached:

Apache
CodeDeploy agent

**→ Create an AMI:**

Actions → Image and Templates → Create Image (AMI)
Include:

Root volume
OS

This AMI becomes your base image for Auto Scaling.

## 5️⃣ Creating a Launch Template

- Go to EC2 → Launch Templates → Create Template
- Use the AMI you just created
- Attach the same IAM role
- Set Auto Scaling Guidance = Enabled
- Save template

After this → Terminate the original EC2 instance
(Your ASG will take over from here.)

## 6️⃣ Create a Load Balancer

- Use Application Load Balancer (ALB):
- Scheme: Internet-facing
- Listeners: HTTP (80)
- Target Group: Register later by ASG
- Health Check Path: /

## 7️⃣ Creating the Auto Scaling Group (ASG)

Go to EC2 → Auto Scaling Groups → Create
Select the Launch Template

- Select 2 AZs
- Attach the Load Balancer Target Group
- Desired Capacity: 2
- Min: 2
- Max: 3

Your ASG will launch 2 EC2 instances from the AMI → both have CodeDeploy agent → both are healthy behind ALB.

## 8️⃣ Setting Up CodeDeploy

- Create Application → Type: EC2/On-premises
- Create Deployment Group:
- Select Service Role (CodeDeploy IAM Role)
- Deployment Type: In-place
- Environment: Amazon EC2 Auto Scaling Group
- Choose your ASG
- Install during deployment: Enable
- Load Balancer: Select Target Group

## 9️⃣ Creating Your App Files on GitHub

Repo structure:

```
/index.html
/appspec.yml

```
index.html

```
<h1>Welcome to my AWS CI/CD Project!</h1>
<p>This deployment was automated using AWS CodeDeploy & CodePipeline.</p>

```
appspec.yml

```
version: 0.0
os: linux

files:
  - source: index.html
    destination: /var/www/html

```

## 🔟 Creating CodePipeline
Now connect the whole workflow:

- Source Stage → GitHub
- Connect to your repo
- Detect changes automatically
- Deploy Stage → CodeDeploy
- Select Application
- Select Deployment Group


![Image description1](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jdd61s71cp5i1msqif8f.png)

Whenever you push changes to the main branch:

Pipeline is triggered
CodeDeploy pushes the latest files
ASG instances receive an update
ALB routes traffic to healthy updated instances


![Image description2](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/5pla0t7n4v83dipqmxdo.png)


![Image description3](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/o37gqd43nb1feyusjzs6.png)

🎉 Full CI/CD and Auto Scaling setup is complete!

#### Deploying a 3-tier architecture in a single AZ on AWS

This document demonstrates how a Three-tier architecture, comprising a Web tier, Application tier, and Database tier, is deployed in a single availability zone on AWS.

First, we create an S3 bucket on AWS with a unique name. This will serve as a codebase for our architecture. Make sure to create this bucket in the same region as your VPC or Lab.

![image](https://github.com/user-attachments/assets/a6a2f58a-9cd6-49e5-a8c8-a840a1389361)

Next, create an IAM role for EC2 and attach AmazonSSMManagedInstanceCore & AmazonS3FullAccess policies for access to S3 and session manager, respectively.

![image](https://github.com/user-attachments/assets/5ade16c6-93e9-4977-a0f5-d8a4b065d1f2)

## Planning your architecture

Below is an architecture diagram of the deployment for reference.

![image](https://github.com/user-attachments/assets/3022075c-9a1c-455a-aa05-0bf53f7d79d4)


Create VPC (keep region consistent)

![image](https://github.com/user-attachments/assets/0de993af-4268-49e1-8f22-bf6cffedd34b)

Create subnets; 3 subnets are required here - Web-PublicSubnet, App-PrivateSubnet, DB-PrivateSubnet

![image](https://github.com/user-attachments/assets/fd9a175b-e79b-4285-ada3-084782457ed2)

![image](https://github.com/user-attachments/assets/39b0ec33-c893-4b8f-bfc8-a121aae4939c)

Next, to give your public subnet internet access, create an internet gateway and attach it to your VPC.

![image](https://github.com/user-attachments/assets/dda64b6c-938e-4a4a-b5ce-01239e1a5cdb)

![image](https://github.com/user-attachments/assets/62ca76a4-01a0-4991-a6eb-658a73dd5996)

Create a NAT GW in the Web Tier for access to the internet by the private subnets.

![image](https://github.com/user-attachments/assets/bfcd2fbe-e795-4a39-bab2-151a127908c6)

Create Public and Private Route tables & associate Webtier with the public subnet, Apptier/DB tier with the private subnet.

![image](https://github.com/user-attachments/assets/e44697d8-529d-46a9-a024-d5c600c32f74)

Edit the routers for the route tables as below. NAT Gateway on the private route table and IGW for the public route table

![image](https://github.com/user-attachments/assets/6b683fc3-b2c8-4cea-b4a0-700271238979)

![image](https://github.com/user-attachments/assets/5c733812-f8db-4e60-8c8c-0a01b1ee5161)












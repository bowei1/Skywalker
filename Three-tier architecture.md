#### Deploying a 3-tier architecture in a single AZ on AWS

This document demonstrates how a Three-tier architecture, comprising a Web tier, Application tier, and Database tier, is deployed in a single availability zone on AWS.

First, we create an S3 bucket on AWS with a unique name. This will serve as a codebase for our architecture. Make sure to create this bucket in the same region as your VPC or Lab.

![image](https://github.com/user-attachments/assets/a6a2f58a-9cd6-49e5-a8c8-a840a1389361)

Next, create an IAM role for EC2 and attach AmazonS3FullAccess policies for access to S3.

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

Create a NAT GW in the Web Tier for access to the internet by the private subnets. Assign an elastic IP.

![image](https://github.com/user-attachments/assets/bfcd2fbe-e795-4a39-bab2-151a127908c6)

Create Public and Private Route tables & associate Webtier with the public subnet, Apptier/DB tier with the private subnet.

![image](https://github.com/user-attachments/assets/e44697d8-529d-46a9-a024-d5c600c32f74)

Edit the routers for the route tables as below. NAT Gateway on the private route table and IGW for the public route table

![image](https://github.com/user-attachments/assets/6b683fc3-b2c8-4cea-b4a0-700271238979)

![image](https://github.com/user-attachments/assets/5c733812-f8db-4e60-8c8c-0a01b1ee5161)

Next, Create Security groups. For the Bastion security group, edit the inbound rule to allow SSH from anywhere and ICMP

![image](https://github.com/user-attachments/assets/9c1ca7e9-c7f1-47b7-aed3-76dfdf5cf6ff)

For Web tier SG, edit inbound rules to allow SSH from VPC, HTTP from anywhere on port 80 and ICMP.

![image](https://github.com/user-attachments/assets/c22f93a3-9488-4737-a7b1-f700b4516b76)

For App tier SG, edit inbound rules to allow SSH from the Security group of Bastion Security group, TCP on port 4000 from webtier security group, amd ICMP.

![image](https://github.com/user-attachments/assets/47951a94-94c1-43fc-a62b-0a3ffaa70404)

For the DB tier SG, edit inbound rules to allow SSH from BastionSG, TCP on port 3306 from the web tier SG, and ICMP.

![image](https://github.com/user-attachments/assets/a66da5f5-f843-4f2c-a16f-6e74c1ae6cca)

Create your instances - Bastion server and Web server in WebTier subnet, app server in Apptier private subnet, and DB server in Db tier private subnet and make sure to assign the EC2s3role created to them.

![image](https://github.com/user-attachments/assets/faa9d6a8-b316-4a08-94dd-977b2212bd7f)

Access your instances in all subnets via the Bastion Host instance.

![image](https://github.com/user-attachments/assets/54459ed3-67c3-4f47-a274-4ba4ebb7178f)


Next step is to install the database on DB server;

```
sudo yum update
sudo dnf install mariadb105-server
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo mariadb
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'place-your-password-here';
CREATE DATABASE dbname;
CREATE TABLE tablename;
    GRANT ALL PRIVILEGES ON database_name.* TO 'your_user_name'@'remote_host' IDENTIFIED BY 'your_password';
    FLUSH PRIVILEGES;
sudo systemctl restart mariadb

sudo mariadb
show databases;
CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL
AUTO_INCREMENT, amount DECIMAL(10,2), description
VARCHAR(100), PRIMARY KEY(id));
INSERT INTO transactions (amount,description) VALUES ('400','groceries');
show tables;
```

Setup the client side of mysql on the Appserver and connect to DB in DBserver

```
sudo yum update
sudo dnf install mariadb105
sudo mariadb -u username -h server-ip/hostname -p
show databases;
```

Clone this repo on your local system ; git clone https://github.com/aws-samples/aws-three-tier-web-architecture-workshop.git

navagate to /aws-three-tier-web-architecture-workshop/application-code/app-tier/Dbconfig.js, fill in your information and save.

![image](https://github.com/user-attachments/assets/d9dc04ee-cdb3-46c2-849c-50f61e9ea032)

Go to s3 and upload the app-tier folder.

![image](https://github.com/user-attachments/assets/d0573c16-d629-48a9-8884-e3673bd12c78)

Now, install backend components(nvm, node.js, pm2) on Appserver

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2
```

Download the code from the S3 bucket onto this server.

```
cd ~/
aws s3 cp s3://BUCKET_NAME/app-tier/ app-tier --recursive
```

Install dependencies and start the app.
```
cd ~/app-tier
npm install
pm2 start index.js
```

To check if the app is running. 
```
pm2 list
```

run the below to keep the app running even after server interruptions;
```
pm2 startup
pm2 save
```

Test the app tier.
```
curl http://localhost:4000/health
curl http://localhost:4000/transaction
```

Next , the web tier. Start by editing the nginx.conf file

![image](https://github.com/user-attachments/assets/7a9a87e5-102b-40e4-9b83-eaef969e0791)

<img width="1467" height="627" alt="image" src="https://github.com/user-attachments/assets/87f98574-942f-4909-896a-27bd4b033a9e" />

Upload this file & application-code/web-tier folder to the s3 bucket.

<img width="1812" height="531" alt="image" src="https://github.com/user-attachments/assets/8c9f562a-262c-447f-9621-48d9bbfcc037" />

Run the following lines of command to install dependencies and Ngnix

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
```

```
cd ~/
aws s3 cp s3://BUCKET_NAME/web-tier/ web-tier --recursive
```

```
cd ~/web-tier
npm install 
npm run build
```

```
sudo dnf update
sudo dnf install nginx -y
```

```
cd /etc/nginx
sudo rm nginx.conf
sudo aws s3 cp s3://BUCKET_NAME/nginx.conf .
sudo service nginx restart
chmod -R 755 /home/ec2-user
sudo chkconfig nginx on
```

When we input the webservr IP, we get the below;

<img width="1862" height="947" alt="image" src="https://github.com/user-attachments/assets/bb337c35-9851-41b0-83cd-2413fc566378" />








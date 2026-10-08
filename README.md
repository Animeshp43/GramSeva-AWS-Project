# GramSeva-Rural-Services-Portal-on-AWS


The objective of this project was to deploy a multi-tier Rural Services Portal on AWS using industry-standard cloud architecture. The application was hosted on EC2 instances, images were stored in Amazon S3, farmer service requests were stored in Amazon RDS MySQL, and the infrastructure was designed to support scalability and high availability.

# **Project Overview**

GramSeva is a cloud-hosted rural services platform where farmers and village families can request services such as **Farming Advice, Tractor on Rent, Soil Check and Animal Doctor** in **English, हिंदी and मराठी**. It was developed using:

•	Frontend: HTML, CSS

•	Backend: PHP 8.2 

•	Web Server: Nginx + PHP-FPM 

•	Database: Amazon RDS MySQL 

<img width="1536" height="1024" alt="GramSeva AWS Architecture Flow" src="https://github.com/user-attachments/assets/8218b020-d0be-4d48-8d8c-40737484bf5e" />


___________________________________________________________________________________________________________________________________________________________________________________________________________________
The project was deployed on AWS using a multi-tier architecture. A custom VPC was designed with public and private subnets across multiple Availability Zones. The application servers were hosted on Amazon EC2, while the database was hosted securely on Amazon RDS in private subnets.

Static website images were stored in Amazon S3 to reduce storage dependency on EC2 instances and improve scalability.

To ensure high availability and fault tolerance, an Amazon Machine Image (AMI) was created from the configured application server and used in a Launch Template. An Auto Scaling Group was configured to automatically maintain healthy instances, while an Application Load Balancer distributed incoming traffic across multiple application servers.

CloudWatch alarms and SNS notifications were configured for infrastructure monitoring and alerting.

# **AWS Services Used**

1. **Networking**

- VPC 
- Public Subnets 
- Private Subnets 
- Route Tables 
- Internet Gateway 
- NAT Gateway 
- Security Groups 
- Availability Zones 

2. **Compute**

- Amazon EC2
- AMI
- Launch Templates
- Auto Scaling Group 

3. **Storage**

- Amazon S3 

4. **Database**

- Amazon RDS MySQL 

5. **High Availability**

- Application Load Balancer 
- Target Groups 

6. **Monitoring**

- CloudWatch
- SNS 

7. **DNS**

- Route 53

<img width="960" height="540" alt="domain" src="https://github.com/user-attachments/assets/35588522-fa39-432d-ab09-e6523f69151a" />

<img width="960" height="539" alt="Request a service" src="https://github.com/user-attachments/assets/93c0cbb5-20d6-4f3f-bd36-93c821815e11" />


___________________________________________________________________________________________________________________________________________________________________________________________________________________
# Step 1 – Create Networking Components
**1. Create VPC**

>Name: my-vpc-virginia

>CIDR: 10.10.0.0/16

>Region: us-east-2

Purpose:

- Creates an isolated network for all AWS resources.

<img width="778" height="279" alt="MYVPC" src="https://github.com/user-attachments/assets/bf55057c-8741-4f17-9844-d180d39d50de" />

**2. Create Subnets**

>**Public Subnet 1** (public-subnet1)

>CIDR: 10.10.1.0/24

>AZ: us-east-1a

>**Public Subnet 2** (public-subnet2)

>CIDR: 10.10.2.0/24

>AZ: us-east-1b

>**Public Subnet 3** (public-subnet3)

>CIDR: 10.10.3.0/24

>AZ: us-east-1c

>**Private Subnet 4** (private-subnet4)

>CIDR: 10.10.4.0/24

>AZ: us-east-1d

Enable **Auto-assign public IPv4 address** on the 3 public subnets.

Purpose:

Public subnets host internet-facing resources.
Private subnet hosts database resources.

<img width="779" height="331" alt="subnets" src="https://github.com/user-attachments/assets/a93b1caa-62c2-4860-bb3d-9dfe87770cb1" />


**3. Create Internet Gateway**

Create IGW

Attach to my-vpc-virginia

Purpose:

- Allows internet access to public resources.

<img width="781" height="208" alt="internet gateway" src="https://github.com/user-attachments/assets/9d750a45-a703-4e0b-917e-fdbeb387560c" />


**4. Create NAT Gateway**

>Name: gramseva-ngw

>Deploy NAT Gateway **in Public Subnet 1**

>Allocate Elastic IP

Purpose:

- Allows private resources to access internet for updates without exposing them publicly.

<img width="777" height="191" alt="nat" src="https://github.com/user-attachments/assets/8051e996-8857-4570-a034-7cdfb6beee29" />


**5. Configure Route Tables**

Create:

>Public Route Table (public-rt)

Route: 

>0.0.0.0/0 → Internet Gateway

Associate with:

>Public Subnet 1
>Public Subnet 2
>Public Subnet 3

then create:

>Private Route Table (private-rt)

Route:

>0.0.0.0/0 → NAT Gateway

Associate with:

>Private Subnet 4

Purpose:

- Controls traffic flow inside VPC.

<img width="778" height="248" alt="route table" src="https://github.com/user-attachments/assets/8a5118e4-3c4b-4a4f-8ffd-236741a342ad" />


# Step 2 – Configure Security Groups

Create:

public-sg (App Servers)

Inbound Rules as
SSH (22) → My IP

HTTP (80) → Anywhere

Outbound as
All Traffic → Anywhere
private-sg (Database)

Inbound Rules as
MYSQL (3306) Source → SG-1
Outbound as
All Traffic

Purpose:

- Only application servers can access database.
- Web servers accept HTTP only from the Load Balancer.
- SSH is allowed only through the Jump Server.

<img width="745" height="42" alt="security group" src="https://github.com/user-attachments/assets/97b2594f-215e-4331-907b-2008d0a75111" />


# Step 3 – Launch EC2 Instances

First Launch:

>Amazon Linux 2023 Jump Server (jump-server)   

inside Public Subnet 1 with security group public-sg and 

With Public IP Enabled

Key pair: jump-server

Purpose:

- Secure SSH access point.

then launch:

>Amazon Linux 2023 Application Server (app-server)

inside Public Subnet 2 with security group public-sg 

Key pair: web-project

Purpose:

- Hosts Nginx and PHP application.

Now Launch DBServer:

>Database Server (database-server) – Amazon Linux 2023 

inside Private Subnet 4 with security group private-sg

with No Public IP

Key pair: database-project

Purpose:

- Used to access private resources (MySQL client to reach RDS).

<img width="778" height="304" alt="instances" src="https://github.com/user-attachments/assets/1870ed76-f279-4945-b34c-46c4330946aa" />


# Step 4 – Configure Application Server

Copy the project files to the app server through the jump server (WinSCP to jump-server, then):

>`ssh -i web.pem ec2-user@<jump-server-public-ip>`

>`chmod 400 app-keypair.pem`

>`ssh -i app-keypair.pem ec2-user@<app-server-private-ip>`

Update packages:

>`sudo update -y`

Install Nginx:

>`sudo install -y nginx`

>`systemctl start nginx`

>`systemctl enable nginx`

Verify:

>`http://<public-ip>`

Install PHP:

>`sudo install -y php8.2 php-fpm php-mysqlnd php-pdo php-mbstring`

>`php -v`

Let PHP-FPM run as the nginx user (prevents 502 errors):

>`systemctl start php-fpm`

>`systemctl enable php-fpm`

>`systemctl restart nginx`

(All the commands above are also available as one script: 

<img width="712" height="739" alt="APPServer" src="https://github.com/user-attachments/assets/e0f56298-ebc2-4da1-b989-fa03deacac29" />


# Step 5 – Upload Website Files

Using WinSCP upload: includes, nginx, public

Move files:

> `mv /home/ec2-user/includes /usr/share/nginx/html`

> `mv /home/ec2-user/public /usr/share/nginx/html`

Set permissions:

> `chown -R nginx:nginx /usr/share/nginx/html/public /usr/share/nginx/html/includes`

> `chmod -R 755 /usr/share/nginx/html/public`

(The `includes` folder stays outside the public web root, so `db_connect.php` can never be opened from a browser.)

![Website Screenshot](screenshots/08-winscp-transfer.png)

# Step 6 – Create Amazon S3 Bucket

>Create Bucket: gramseva-project-bucket-yourname  (Region: us-east-2)

Configuration:

- ACL Enabled
- Public Access Allowed (untick "Block all public access")
- Versioning Disabled

Upload:

- images/

Add this bucket policy so only the images are public:

    {
      "Version": "2012-10-17",
      "Statement": [{
        "Sid": "PublicReadImages",
        "Effect": "Allow",
        "Principal": "*",
        "Action": "s3:GetObject",
        "Resource": "arn:aws:s3:::gramseva-project-bucket-yourname/images/*"
      }]
    }

Purpose:

- Stores static images.
- Reduces EC2 storage usage.

![Website Screenshot](screenshots/09-s3-bucket.png)

# Step 7 – Update Application Image URLs

Edit:

> `vim /usr/share/nginx/html/public/index.html`

> `vim /usr/share/nginx/html/public/request.php`

Replace old S3 bucket URLs with your bucket URL (in vim):

    :%s/gramseva-demo-bucket/gramseva-project-bucket-yourname/g

Update all image references (logo + 4 service images).

# Step 8 – Create Amazon RDS MySQL

Create DB Subnet Group:

>Name: gramseva-subnet-group

>Subnets: private-subnet4 and private-subnet5

Create Database:

>Engine: MySQL

    DB Identifier: gramseva-db
    Username: admin
	Password: ********

Connectivity:

- Same VPC
- Private Access (Public access: No)
- Subnet Group: gramseva-subnet-group
- Security Group: db-sg

Disable:

- Automatic Backups
- Enhanced Monitoring
- Maintenance Window

Launch RDS and copy the **endpoint** once the status is Available.

![Website Screenshot](screenshots/10-rds-created.png)

# Step 9 – Configure Database Connection

Create the real config file from the example (this file is never committed to GitHub):

>`cd /usr/share/nginx/html/includes`

>`cp db_connect.example.php db_connect.php`

>`vim db_connect.php`

Update:

    $servername = "gramseva-db.xxxxx.us-east-2.rds.amazonaws.com";
    $username = "admin";
    $password = "YOUR_STRONG_PASSWORD";
    $dbname = "gramseva_db";

Save and exit.

# Step 10 – Create Database Tables

Connect from DB Server (login through the jump server):

>`ssh -i db-keypair.pem ec2-user@<database-server-private-ip>`

Install MySQL Client:

>`sudo sudo install -y mariadb105`

Connect:

    mysql -h <RDS-ENDPOINT> -u admin -p

Create database:

    CREATE DATABASE gramseva_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    USE gramseva_db;
    Create table:

>CREATE TABLE service_requests(

    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    contact VARCHAR(20) NOT NULL,
    village VARCHAR(100) NOT NULL,
    email VARCHAR(150) NULL,
    service_type ENUM('Farming Advice','Tractor on Rent','Soil Check','Animal Doctor') NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'New',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP

);

(Or simply run the ready-made file: `mysql -h <RDS-ENDPOINT> -u admin -p < database/schema.sql`)

Verify:

    SHOW TABLES;
	DESC service_requests;

![Database Entries](screenshots/11-db-table.png)

# Step 11 – Configure Domain

Create hostname using No-IP.

Example:

gramseva.hopto.org

Point domain to:

App Server Public IP (for testing), then Application Load Balancer DNS (after Step 17)

Purpose:

- Access application using a friendly URL.

![Website Screenshot](screenshots/12-domain-noip.png)

# Step 12 – Configure Nginx Virtual Host

Edit:

>`vim /home/ec2-user/nginx/gramseva.conf`

Example:

>server {

>listen 80 default_server;

>server_name gramseva.hopto.org;

>root /usr/share/nginx/html/public;

>index index.html index.php;

>location / {

>try_files $uri $uri/ /index.php?$query_string;}

>location ~ \.php$ {

>include fastcgi.conf;

>fastcgi_pass unix:/run/php-fpm/www.sock;}

>}

Move file:

>`mv /home/ec2-user/nginx/gramseva.conf /etc/nginx/conf.d/`

Validate:

>`nginx -t`

Restart:

>`systemctl restart nginx`

>`systemctl reload nginx`

(`default_server` lets the site answer on the Load Balancer DNS name and IP address too.)

# Step 13 – Test Application

Open:

>`http://gramseva.hopto.org`

Switch the language: **English / हिंदी / मराठी**

Click **Request a Service**, submit the form.

Verify data inside RDS:

>`SELECT * FROM service_requests;`

![Website Screenshot](screenshots/16-request-form.png)
![Database Entries](screenshots/17-db-entries.png)

# Step 14 – Create AMI

From working App Server:

Actions

→ Image and Templates

→ Create Image

Name:

>gramseva-app-ami

Purpose:

- Creates reusable application image.

![Website Screenshot](screenshots/18-ami.png)

# Step 15 – Create Launch Template

Name: gramseva-lt

Use:

gramseva-app-ami

Configure:

Instance Type (t3.micro)

Security Group (web-sg)

Key Pair (app-keypair)

Purpose:

- Standard template for Auto Scaling.

![Website Screenshot](screenshots/19-launch-template.png)

# Step 16 – Create Target Group

>Name: gramseva-tg

>Type: Instance

>Protocol: HTTP

>Port:80

>Health check path: /

Register application instances.

![Website Screenshot](screenshots/20-target-group.png)

# Step 17 – Create Application Load Balancer

Name: gramseva-alb

Configure: Internet Facing, Public Subnets 1, 2 and 3 with HTTP Listener (80)

Security Group: alb-sg

Attach: Target Group (gramseva-tg)

Purpose:

- Distributes traffic across multiple servers.

![Website Screenshot](screenshots/21-load-balancer.png)

# Step 18 – Create Auto Scaling Group

Name: gramseva-asg

Use Launch Template (gramseva-lt).

Subnets: public-subnet2 and public-subnet3

Configuration:

>Desired Capacity: 2

>Minimum: 2

>Maximum: 4

>Scaling policy: Target tracking – Average CPU 50%

Attach:

>Application Load Balancer (gramseva-tg) with ELB health checks ON

Purpose:

- Automatically adds/removes instances.

![Auto Scaling](screenshots/22-auto-scaling.png)

# Step 19 – Monitoring
1. CloudWatch

Create alarms:

>CPU > 70%  (gramseva-high-cpu)

>Unhealthy hosts >= 1  (gramseva-unhealthy-hosts)

>RDS CPU > 80%  (gramseva-rds-cpu)

![Auto Scaling](screenshots/23-cloudwatch-alarms.png)

2. SNS

Create topic: gramseva-alerts

Subscribe email.

Purpose:

- Sends notifications when alarms trigger.

![Auto Scaling](screenshots/24-sns-notification.png)

_____________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________________
# Architecture Outcome

**The final architecture provides:**

- ✅ Secure network isolation using VPC
- ✅ Public and private subnet segregation
- ✅ Centralized access through Jump Server
- ✅ Scalable application layer using Auto Scaling
- ✅ High availability through ALB and Multi-AZ deployment
- ✅ Managed database using Amazon RDS MySQL
- ✅ Static content delivery through Amazon S3
- ✅ Monitoring and alerting using CloudWatch and SNS
- ✅ Farmer-friendly multilingual website (English, हिंदी, मराठी)

**This design closely resembles a real-world production environment and demonstrates core AWS Infrastructure, Networking, Security, High Availability, and Monitoring concepts.
**

___________________________________________________________________________________________________________________________________________________________________________________________________________________
# More Documentation

- [Project Execution Flow](docs/execution-flow.md)
- [Project Outcome](docs/project-outcome.md)
- [Full Deployment Guide](docs/deployment-guide.md)
- [Troubleshooting](docs/troubleshooting.md)
- [Clean-up (avoid AWS charges)](docs/cleanup.md)

**Author:** YOUR NAME  |  [GitHub](https://github.com/your-username)  |  [LinkedIn](https://www.linkedin.com/in/your-profile)

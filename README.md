# project-15
#### AWS CLOUD SOLUTION FOR 2 COMPANY WEBSITES USING A REVERSE PROXY TECHNOLOGY
You will build a secure infrastructure inside AWS VPC (Virtual Private Cloud) network for a fictitious company (Choose an interesting name for it) that uses WordPress CMS for its main business website, and a Tooling Website (https://github.com/<your-name>/tooling) for their DevOps team. As part of the company’s desire for improved security and performance, a decision has been made to use a reverse proxy technology from NGINX to achieve this.

Cost, Security, and Scalability are the major requirements for this project. Hence, implementing the architecture designed below, ensure that infrastructure for both websites, WordPress and Tooling, is resilient to Web Server’s failures, can accommodate to increased traffic and, at the same time, has reasonable cost.
![](https://github.com/UzonduEgbombah/project-15/assets/137091610/47615495-93c8-45ec-9237-3d81133c58e6)

#### SET UP A VIRTUAL PRIVATE NETWORK (VPC)
- Create a VPC

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/4c4b4ede-9fa5-47cb-b8bd-fe1f5d9ce5b7)

- Create subnets as shown in the architecture

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/bca44d77-10ac-4388-864e-94348ebc6a49)
 
- Create a route table and associate it with public subnets and  private subnets

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/89ac2c3b-e10e-4bde-8357-826e73467b32)

- Create an Internet Gateway

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/534f65bf-b5ee-45eb-8178-6cd948142ada)

- Edit a route in public route table, and associate it with the Internet Gateway. (This is what allows a public subnet to be accisble from the Internet)

- Create 3 Elastic IPs

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/daf48833-92f6-4368-9932-7227ee287560)


- Create a Nat Gateway and assign one of the Elastic IPs (*The other 2 will be used by Bastion hosts)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/76ee5e2d-ee1e-4da6-a2a3-2cd3150f62a9)

#### Create a Security Group for:

###### Nginx Servers:
Access to Nginx should only be allowed from a Application Load balancer (ALB). At this point, we have not created a load balancer, therefore we will update the rules later. For now, just create it and put some dummy records as a place holder.
###### Bastion Servers:
Access to the Bastion servers should be allowed only from workstations that need to SSH into the bastion servers. Hence, you can use your workstation public IP address. To get this information, simply go to your terminal and type curl www.canhazip.com
Application Load Balancer: ALB will be available from the Internet
###### Webservers:
Access to Webservers should only be allowed from the Nginx servers. Since we do not have the servers created yet, just put some dummy records as a place holder, we will update it later.
Data Layer: Access to the Data layer, which is comprised of Amazon Relational Database Service (RDS) and Amazon Elastic File System (EFS) must be carefully desinged – only webservers should be able to connect to RDS, while Nginx and Webservers will have access to EFS Mountpoint.

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/c070def8-7f85-4a20-8f15-945b33813bda)

#### Proceed With Compute Resources
You will need to set up and configure compute resources inside your VPC. The recources related to compute are:

- EC2 Instances
- Launch Templates
- Target Groups
- Autoscaling Groups
- TLS Certificates
- Application Load Balancers (ALB)

#### Set Up Compute Resources for Nginx
Provision EC2 Instances for Nginx
Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) in any 2 Availability Zones (AZ) in any AWS Region (it is recommended to use the Region that is closest to your customers). Use EC2 instance of T2 family (e.g. t2.micro or similar)

Ensure that it has the following software installed:

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop

Create an AMI out of the EC2 instance

Prepare Launch Template For Nginx (One Per Subnet)
Make use of the AMI to set up a launch template
Ensure the Instances are launched into a public subnet
Assign appropriate security group
Configure Userdata to update yum package repository and install nginx
Configure Target Groups
Select Instances as the target type
Ensure the protocol HTTPS on secure TLS port 443
Ensure that the health check path is /healthstatus
Register Nginx Instances as targets
Ensure that health check passes for the target group
Configure Autoscaling For Nginx
Select the right launch template
Select the VPC
Select both public subnets
Enable Application Load Balancer for the AutoScalingGroup (ASG)
Select the target group you created before
Ensure that you have health checks for both EC2 and ALB
The desired capacity is 2
Minimum capacity is 2
Maximum capacity is 4
Set scale out if CPU utilization reaches 90%
Ensure there is an SNS topic to send scaling notifications

#### Process and command for NGINX will be shown below :

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/4fb3fe32-e0c1-40f6-81b7-2b618d5e75e4)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/d40bf695-4016-4830-8478-7d6823306e9e)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/534b2536-38f1-4248-8a27-250ae5f1b175)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/a8b2ce3e-5b64-41d9-8fc6-bce542f18410)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/6021f93d-4a6d-4a3c-972c-a416fd934c09)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/18379d81-02f3-424b-89f4-ed1955b679d9)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/88a43c72-7aea-4a91-87a3-1c129cb4056d)


#### Set Up Compute Resources for Bastion
Provision the EC2 Instances for Bastion
Create an EC2 Instance based on CentOS Amazon Machine Image (AMI) per each Availability Zone in the same Region and same AZ where you created Nginx server
Ensure that it has the following software installed

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
  
Associate an Elastic IP with each of the Bastion EC2 Instances
Create an AMI out of the EC2 instance
Prepare Launch Template For Bastion (One per subnet)
Make use of the AMI to set up a launch template
Ensure the Instances are launched into a public subnet
Assign appropriate security group
Configure Userdata to update yum package repository and install Ansible and git
Configure Target Groups
Select Instances as the target type
Ensure the protocol is TCP on port 22
Register Bastion Instances as targets
Ensure that health check passes for the target group
Configure Autoscaling For Bastion
Select the right launch template
Select the VPC
Select both public subnets
Enable Application Load Balancer for the AutoScalingGroup (ASG)
Select the target group you created before
Ensure that you have health checks for both EC2 and ALB
The desired capacity is 2
Minimum capacity is 2
Maximum capacity is 4
Set scale out if CPU utilization reaches 90%
Ensure there is an SNS topic to send scaling notifications

#### Process and command for BASTION will be shown below :

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/9d7e4e33-a582-4dc0-bf1d-dc17e55a572a)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/0418cf29-3883-407e-a0c5-813dcf3b14a6)







#### Set Up Compute Resources for Webservers
Provision the EC2 Instances for Webservers
Now, you will need to create 2 separate launch templates for both the WordPress and Tooling websites

Create an EC2 Instance (Centos) each for WordPress and Tooling websites per Availability Zone (in the same Region).

Ensure that it has the following software installed

- python
- ntp
- net-tools
- vim
- wget
- telnet
- epel-release
- htop
- php
- Create an AMI out of the EC2 instance

Prepare Launch Template For Webservers (One per subnet)
Make use of the AMI to set up a launch template
Ensure the Instances are launched into a public subnet
Assign appropriate security group
Configure Userdata to update yum package repository and install wordpress (Only required on the WordPress launch template)

#### Process and command for WEBSERVERS will be shown below :

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/f40b439f-d8b6-4ef7-886b-35252d083ef1)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/ffa38c23-acc0-4f4c-ada4-fad89e28f82d)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/fa81f81c-0c41-4bde-b7b0-12c73968cd70)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/8c568e42-1346-4653-8073-d58528dac492)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/fe152bb6-1ec3-48d2-890a-b38abf062150)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/ce6fbdb5-0212-4cf4-b0d5-c3cac50b9952)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/84b1ad42-5d8a-4084-8d86-069499b53ea5)


#### TLS Certificates From Amazon Certificate Manager (ACM)
You will need TLS certificates to handle secured connectivity to your Application Load Balancers (ALB).

- Navigate to AWS ACM
- Request a public wildcard certificate for the domain name you registered in Freenom
- Use DNS to validate the domain name
- Tag the resource

#### AWS Certified Manager

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/4f486220-7788-48b1-876d-c69adbc53038)  


## CONFIGURE APPLICATION LOAD BALANCER (ALB)
Application Load Balancer To Route Traffic To NGINX
Nginx EC2 Instances will have configurations that accepts incoming traffic only from Load Balancers. No request should go directly to Nginx servers. With this kind of setup, we will benefit from intelligent routing of requests from the ALB to Nginx servers across the 2 Availability Zones. We will also be able to offload SSL/TLS certificates on the ALB instead of Nginx. Therefore, Nginx will be able to perform faster since it will not require extra compute resources to valifate certificates for every request.

- Create an Internet facing ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select Nginx Instances as the target group
- Application Load Balancer To Route Traffic To Web Servers
Since the webservers are configured for auto-scaling, there is going to be a problem if servers get dynamically scalled out or in. Nginx will not know about the new IP addresses, or the ones that get removed. Hence, Nginx will not know where to direct the traffic.

To solve this problem, we must use a load balancer. But this time, it will be an internal load balancer. Not Internet facing since the webservers are within a private subnet, and we do not want direct access to them.

- Create an Internal ALB
- Ensure that it listens on HTTPS protocol (TCP port 443)
- Ensure the ALB is created within the appropriate VPC | AZ | Subnets
- Choose the Certificate from ACM
- Select Security Group
- Select webserver Instances as the target group
- Ensure that health check passes for the target group
NOTE: This process must be repeated for both WordPress and Tooling websites.

#### NOTE:
some of the process below may have been defined earlier in the documentation.

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/5f63cd89-4ab4-42ef-9197-31501bb15451)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/d291d2a7-5cea-4496-865d-4596a6408a16)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/66438085-b8d2-45b3-8b38-a41c09321ea0)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/70cd7cb0-3b26-40bb-9a6c-3cf8b9aa1acd)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/5605df74-5cc3-4b2a-b41f-f21fefd72a64)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/90f08d52-35a2-4759-a7d5-e8612f94b5a2)

#### Setup EFS
Amazon Elastic File System (Amazon EFS) provides a simple, scalable, fully managed elastic Network File System (NFS) for use with AWS Cloud services and on-premises resources. In this project, we will utulize EFS service and mount filesystems on both Nginx and Webservers to store data.

- Create an EFS filesystem
- Create an EFS mount target per AZ in the VPC, associate it with both subnets dedicated for data layer
- Associate the Security groups created earlier for data layer.
- Create an EFS access point. (Give it a name and leave all other settings as default)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/0961dc7b-e0fb-4599-8dcb-e938994da940)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/2157a540-b775-41bf-82e1-35a6b9eccbb3)

#### Setup RDS
Pre-requisite: Create a KMS key from Key Management Service (KMS) to be used to encrypt the database instance.

Amazon Relational Database Service (Amazon RDS) is a managed distributed relational database service by Amazon Web Services. This web service running in the cloud designed to simplify setup, operations, maintenans & scaling of relational databases. Without RDS, Database Administrators (DBA) have more work to do, due to RDS, some DBAs have become jobless

To ensure that yout databases are highly available and also have failover support in case one availability zone fails, we will configure a multi-AZ set up of RDS MySQL database instance. In our case, since we are only using 2 AZs, we can only failover to one, but the same concept applies to 3 Availability Zones. We will not consider possible failure of the whole Region, but for this AWS also has a solution – this is a more advanced concept that will be discussed in following projects.

#### To configure RDS, follow steps below:

- Create a subnet group and add 2 private subnets (data Layer)
- Create an RDS Instance for mysql 8.*.*
To satisfy our architectural diagram, you will need to select either Dev/Test or Production Sample Template. But to minimize AWS cost, you can select the Do not create a standby instance option under Availability & durability sample template (The production template will enable Multi-AZ deployment)
Configure other settings accordingly (For test purposes, most of the default settings are good to go). In the real world, you will need to size the database appropriately. You will need to get some information about the usage. If it is a highly transactional database that grows at 10GB weekly, you must bear that in mind while configuring the initial storage allocation, storage autoscaling, and maximum storage threshold.
- Configure VPC and security (ensure the database is not available from the Internet)
- Configure backups and retention
- Encrypt the database using the KMS key created earlier
- Enable CloudWatch monitoring and export Error and Slow Query logs (for production, also include Audit)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/38ee87d8-9c9d-443a-a32d-6a17eb6e7d70)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/0213ddca-a485-4b47-9fec-eef7cc6511f4)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/291bbc11-c487-4e43-83f2-b317d5988ce5)

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/82f8058c-74d6-4d8f-b936-87c45f182883)

#### Note This service is an expensinve one. Ensure to review the monthly cost before creating. (DO NOT LEAVE ANY SERVICE RUNNING FOR LONG)

#### Configuring DNS with Route53
Earlier in this project you registered a free domain with Freenom and configured a hosted zone in Route53. But that is not all that needs to be done as far as DNS configuration is concerned.

You need to ensure that the main domain for the WordPress website can be reached, and the subdomain for Tooling website can also be reached using a browser.

Create other records such as CNAME, alias and A records.

NOTE: You can use either CNAME or alias records to achieve the same thing. But alias record has better functionality because it is a faster to resolve DNS record, and can coexist with other records on that name. Read here to get to know more about the differences.

Create an alias record for the root domain and direct its traffic to the ALB DNS name.
Create an alias record for tooling.<yourdomain>.com and direct its traffic to the ALB DNS name.

![](https://github.com/UzonduEgbombah/project-15/assets/137091610/49a5bf24-a21e-45ba-8e3a-2ee3374834f5)



## Create 3-tier architecture manually in AWS first 

A three-tier architecture is a software architecture pattern where the application is broken down into three logical tiers:

- **The presentation layer** (display and collect info from the user and send its content to the browser in the form of HTML/JS/CSS)
- **The business logic/application layer** (Prescribes how business objects interact with each other and handles collected information from the presentation tier. It can also add, delete or modify data in the database tier)
- **The data storage layer** (data or backend tier of a web application. It is where the information processed by the application tier is stored and managed)



#### Service components 
I will be using these AWS service components to design and build a three-tier cloud infrastructure:

- **Virtual Private Cloud (VPC)** - virtual network where you can create and manage your AWS resource in a secure and scalable way.
- **Internet Gateway** - the IG allows communication between EC2 instance in the VPC and the internet. The VPC needs to attached to the internet gateway
- **Subnets** - The subnet is a way for us to group our resources within the VPC with their IP range. A subnet can be public or private. EC2 instances within a public subnet have public IPs and can directly access the internet while those in the private subnet does not have public IPs and can only access the internet through a NAT gateway.
- **NAT Gateway** - The NAT gateway enables the EC2 instances in the private subnet to access the internet. The NAT Gateway is an AWS managed service for the NAT instance.
- **Route tables** - Route tables is a set of rule that determines how data moves within our network. We need two route tables; private route table and public route table. The public route table will define which subnets that will have direct access to the internet ( ie public subnets) while the private route table will define which subnet goes through the NAT gateway (ie private subnet). The public and the private subnet needs to be associated with the public and the private route table respectively. We also need to route the traffic to the internet through the **internet gateway** for our public route table.
- **Auto Scaling Group** - Auto Scaling Group can automatically adjust the size of the EC2 instances serving the application based on need. This is what makes it a good approach rather than directly attaching the EC2 instances to the load balancer.


#### How I approached it 
##### Setting up the Network
1. **Create the VPC** (name and CIDR block of 10.0.0.0/16)
```ruby
Navigate to VPC → Create VPC →VPC Settings → Resources to create → VPC only (include a name), IPV4 CIDR block → 10.0.0.0/16 → Create VPC
```

2. **Create the Subnets** (2 for each layer)
```ruby
Subnets → Create subnet → VPC → VPC ID → select the VPC created in the previous step. 
Name each subnet, choose the availability zone provide a CIDR for each.

Name: public web subnet 1
AZ: us east 1a
IPv4 CIDR block: 10.0.16.0/20

Name: public web subnet 2
AZ: us east 1b
IPv4 CIDR block: 10.0.0.0/20

Name: private app subnet 1
AZ: us east 1a
IPv4 CIDR block: 10.0.144.0/20

Name: private app subnet 2
AZ: us east 1b
IPv4 CIDR block: 10.0.176.0/20

Name: private database subnet 1
AZ: us east 1a
IPv4 CIDR block: 10.0.128.0/20

Name: private database subnet 2
AZ: us east 1b
IPv4 CIDR block: 10.0.160.0/20
```

3. **Create the Internet Gateway** 
```ruby
Create Internet Gateway (include a name) → Attach it to the VPC → Select the gateway, click on the arrow next to Actions, and choose Attach to VPC
```

4. **Create the NAT Gateway** (to allow private subnets to connect to the internet)
```ruby
- Create NAT gateway → Provide a name (optional), specify a public subnet and select Public for Connectivity type.

- Select “Allocate Elastic IP” to generate an IP address that will serve as a replacement for the source IP of the instances and translate addresses back to the source IPs.
```

5. **Create Route tables** (public and private)
```ruby
- Create two route tables (public and private)
- Add name and attach VPC 

- Add routes to the two route tables to allow them to communicate inside or outside the VPC 
-- For the public route table, add your CIDR route (10.0.0.0/16) and target local. 
Also add a route and enter the Destination (0.0.0.0/0) and select Internet Gateway
-- For the private route table, add a route and enter the Destination (0.0.0.0/0) and select NAT Gateway


- Associate the  subnets with the route tables in order to allow them to communicate via the route table. 
-- In the public route table, select Subnet Associations, select Edit subnet associations, select the 2 public subnets you created and select Save changes.

-- In the private route table, select Subnet Associations, select Edit subnet associations, select the 2 private subnets you created and select Save changes.
```


##### Creating EC2 Instances for presentation and application tier 

Creating an EC2 Instance with an Auto scaling groups in order to connect to our static website in the presentation tier 

Creating an EC2 Instance for the presentation tier

**Create Launch template for presentation tier** (public facing)
1. -- Launch template name and description
2. -- Application and OS Images (Amazon Machine Image) — free-tier AMI
3. -- Instance type (t.2 micro)
-- Key pair (create a key pair and download to device)
4. -- Network settings (create security group with name and description, select vpc, add Security group rule (Type: ssh Source: 0.0.0.0/0, add Security group rule (Type: http Source: 0.0.0.0/0))
5. -- Advanced details (add script to use data for the static website)
'''ruby
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl start httpd
sudo systemct enable httpd
'''
6. -- Create template 


Creating an EC2 Instance for the application tier

**Create Launch template for application tier** (private facing)
1. -- Launch template name and description
2. -- Application and OS Images (Amazon Machine Image) — free-tier AMI
3. -- Instance type (t.2 micro)
-- Key pair (create a key pair and download to device)
4. -- Network settings (create security group with name and description, select vpc, add Security group rule (Type: ssh Source: nameofpresentationtiersecuritygroup)
5. -- Create template 


**Create Auto scaling groups** (to balance the load in the tiers)
1. -- Name of presentation tier ASG
2. -- Select launch template for presentation tier and the VPC (select both public subnets)
3. -- Click Enable group metrics collection within CloudWatch
4. -- Group size
- Desired — 2
- Minimum — 2
- Maximum — 3
5. -- Scaling Policies
- Click Target tracking scaling policy
- Target value — 80 and select Skip to Review

You can create an ASG for the application tier but use the launch template for the application tier and the VPC (2 private subnets)


##### Creating database tier
Search and select RDS from the AWS console. From the left side of the screen, select Subnet groups and select Create DB Subnet group.

**Subnet group details**
- Name your subnet group and a description
- Select your VPC you created earlier
**Add Subnets**
- Select your availability zones we used eariler(us-east-1a and us-east-1b)
- Select the private db subnets we created eariler, and add the same CIDR we used and select Create

Select Database and Create Database:
- Choose a database creation method (Standard)
- Engine options (Select My SQL)
- Templates (Free tier)
- Settings
DB Cluster Identifier (privatedb)
Master password (create a password) and confirm it
- Storage
uncheck Enable storage autoscaling
- Connectivity
Select the VPC we created eariler
VPC Security group (select Create new)
New VPC security group name ()
Create database

Now we will connect the Application tier and the Database tier. Once your database is created, select it and navigate to Connectivity & Security. Under security click on VPC Security Groups link. Select inbound rules along the bottom and Edit inbound rule. We will add rule to allow traffic from the application tier and delete the existing rule. In the port you will want to enter 3306 and our source will be the **nameoftheprivateappsecuritygroup** we created earlier.

To check if we are set up correctly, ssh into one of my Public Web ec2 instances.
First I pinged the Private App instance from my Public Web instance. I did this by running the command
'''ruby 
ping <private IP address of private app instance>
'''

Try to access the database from the Public Web instance
'''ruby 
ping <RDS_Endpoint>
'''
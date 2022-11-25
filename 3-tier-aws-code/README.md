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
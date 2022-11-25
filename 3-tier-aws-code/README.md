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

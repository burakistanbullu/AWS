# IAM General Information

What is IAM?

Identity and Access Management (IAM) is a service that controls authentication (who can access) and authorization (what they can do).

When creating a user, you provide authentication details, and by assigning groups and policies you handle the authorization part.

## Basic Concepts

### User

**What is a User in IAM?**

A user represents a real person who needs access to AWS.
When you create an IAM user, AWS provides a link like:
https://<-account-id->.signin.aws.amazon.com/console

Your AWS Account ID is part of this link (e.g., 554435643459) and when logging into the AWS console, you use ; Username, Password and that Account ID.

**Types of Users**

There are two types of users in AWS:

**1. Root User**
-   Created automatically when the AWS account is first set up.
-   Has unrestricted access to all AWS resources.
-   IAM policies cannot restrict the root user.
-   Security audits will highlight root login events as critical or unsafe.
-   Only one root user exists per AWS account.

**2. IAM User**

-   A regular user created by the root user or an admin.
-   Permissions are restricted using IAM policies.
-   Used for day-to-day operations.

**User Management Best Practices**

*AWS strongly recommends:*

“Use the root user only for account and administrative tasks.
Create IAM users for daily operational work.”

*In other words:*

-   Avoid logging in with the root user unless absolutely necessary.
-   Create an AdministratorAccess IAM user for admin operations (example: burak-admin).
-   Enable MFA (Multi-Factor Authentication) for the root user.
-   Avoid generating or using root access keys.
-   Do not use the root user for regular tasks.

### Policy,Group,Role
#### IAM Policy

-   Policies are JSON-based documents defining:
-   Who can do what actions under which conditions?
-   Policies can be attached to: Users, Groups, Roles

#### IAM Group

-   A group is a collection of users.
-   Instead of assigning policies to each user individually, you assign policies to a group and place users inside it.
-   This simplifies permission management, especially in large environments.

#### IAM Role

-   A role is used when a service, application, or external identity needs access to AWS resources.

- Roles do not have permanent credentials (no username/password).

- When an application assumes a role;
    
    -   AWS generates temporary security credentials (access key, secret key, session token).

    -   The application uses these credentials to interact with AWS securely.

# Region

**What is a Region?**

An AWS region is a *geographical area* where AWS data centers are located.

-   Each region is a *completely independent* AWS infrastructure.

-   Example: us-east-1 → North Virginia (USA), eu-west-1 → Ireland

An AWS region code is structured like this ➡ continent-direction-number 
➡ us-east-1

Each region contains its own resources (EC2, S3, RDS, etc.).
-   Meaning: An EC2 instance you create in Frankfurt cannot automatically access an S3 bucket in Virginia unless you explicitly allow it.

**Factors Affecting Region Selection**

When choosing an AWS region, you should consider four key factors:

-   *Latency* : Select a region close to your users for better performance.

-  *Compliance / Data Residency* : Some data must stay within certain countries (e.g., GDPR).

-   *Service Availability* : Not every AWS service is available in every region (e.g., some AI services are only available in the US).

-   *Cost* : Prices vary by region (e.g., Virginia is generally cheaper).

# Availability Zone

**What is an Availability Zone (AZ)?**

A region consists of multiple Availability Zones (AZs). *These are physical data centers within the same region.*

-   A region = like a city
-   AZs = separate data centers within that city

Example: eu-central-1 (Frankfurt) Region
-   Contains AZs such as eu-central-1a and eu-central-1b.

Each Availability Zone:
-   Has its own power, cooling, and network infrastructure.
-   Is connected to other AZs with low-latency fiber networks.
-   Enables High Availability by allowing deployments across multiple AZs.

# EC2 (Elastic Compute Cloud)

## What is EC2? Why Do We Use It? What Are EC2 Instance Types?

**What is EC2?**

The name EC2 comes from the initials of “Elastic Compute Cloud.”
It is AWS’s virtual server (virtual machine) service.

If an AWS service includes the word “elastic,” it means that the service is scalable.

**Why Do We Use EC2?**

-   *To reduce management overhead* ➜ OS upgrades, security issues, new server provisioning, datacenter maintenance and operations.

-   *To lower costs* ➜ With the pay-as-you-go model, we can shut down servers when they are not in use and reduce expenses (e.g., shutting down batch servers at night).

**EC2 Instance Types**

-   General (Balanced CPU-memory) ➜ Suitable for enterprise applications (e.g., payment systems).
-   Compute Optimized (CPU-intensive) ➜ Suitable for CI/CD runners (Jenkins) and compute-heavy applications.
-   Memory Optimized (High Memory) ➜ Suitable for caching (Redis) and database workloads.
-   Accelerated Computing (GPU) ➜ Suitable for machine learning applications.
-   Storage Optimized (SSD/IOPS) ➜ Suitable for log processing (Elasticsearch), high-performance databases (Cassandra), and Big Data analytics (Hadoop, Spark).

## Creating an EC2 Instance, Installing Jenkins, and Accessing It from the Outside World

**Creating an EC2 Instance**

-   On the EC2 / Instances page, click “Launch Instances.”
-   Fill out the required fields on the screen:
    -   Name: instance name
    -   OS image: choose operating system (we selected Ubuntu)
    -   Instance type: CPU, RAM resources of your server
    -   Key pair (login): Click “Create new key pair”, give a name, keep defaults (RSA and .pem), and click Create key pair. The private key will be downloaded to your computer.
        -   What is this?
            -   It’s the combination of private + public key we use to connect to the instance.
            -   Public key is stored by AWS, private key (.pem) is downloaded to your machine. This is why the private key must never be shared.

    -   Click “Launch Instance” to create the instance.
-   To connect, take the Public IP of the instance and run from your computer:

    ```bash
    ssh -i .\aws_login.pem ubuntu@13.61.179.67
    ```


-   Note : (Here, aws_login.pem is the private key downloaded to your computer, and ubuntu is Ubuntu’s default user.)

**Installing Jenkins on the Server**

1. Install Java JDK
    ```bash
    sudo apt update
    sudo apt install fontconfig openjdk-21-jre
    java --version
    ```

2. Install Jenkins
    ```bash
    sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
    https://pkg.jenkins.io/debian/jenkins.io-2023.key

    echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] \
    https://pkg.jenkins.io/debian binary/" | sudo tee \
    /   etc/apt/sources.list.d/jenkins.list > /dev/null

    sudo apt update
    sudo apt install jenkins
    ```

3. Check if the service is running:
    ```bash
    systemctl status jenkins
    ```

**Accessing Jenkins from the Outside World**

-   By default, an instance has no inbound access except via SSH.
Jenkins runs on port 8080, so we must add an Inbound Rule.

-   In the AWS console, click your instance → go to the Security tab → open the Security groups.

-   Under Inbound rules, click Edit inbound rules and add:

    -   Type: Custom TCP
    -   Port range: 8080
    -   Source: Anywhere

-   Test access in your browser: http://13.61.179.67:8080/


# VPC (Virtual Private Cloud)

**What is a VPC?**

A VPC is a private network within AWS.
-   It is like having a dedicated section inside AWS’s massive data centers — a private data center allocated just for you.
-   You can think of it as an isolated virtual network inside AWS infrastructure.

When creating a VPC, we define an IP range, which is called a CIDR Block.

## What is Subnet ve CIDR ? Public & Private Subnet 

**What is an IP Range (CIDR Block) and What is a Subnet?**

The first step in creating a VPC is *defining an IP range (CIDR).* Example: 10.0.0.0/16
-   This means a private IP space containing 65,536 IP addresses
(32 – 16 = 16 → 2^16 = 65,536)
    -   This means we can assign this many IPs to resources in our project.

-   We can divide this IP range into smaller sections. These sections are called *Subnets*. Example subnetting:
    -   *10.0.1.0/24 → Frontend subnet*
    -   *10.0.2.0/24 → Database subnet*

-   These are now subnets. All of them belong to the same VPC but represent *different sub-networks.*

**What is a Public Subnet vs Private Subnet?**

What makes a subnet *public* or *private* is not the IP range — it is the *routing rules.*
-   We classify subnets as Public or Private depending on their access to the internet.

*Public Subnet*
-   A public subnet has a route to:
    -   0.0.0.0/0 → Internet Gateway
-   This allows:
    -   Instances in the subnet to access the internet (if they have a public IP).
    -   These instances to be reachable from the internet.

*Private Subnet*

-   A private subnet does NOT access the internet directly.
-   Instances inside do not have a route to the Internet Gateway.
-   If they need internet access, they go via a NAT Gateway located inside a public subnet.
-   Routing rules:
    -   0.0.0.0/0 → NAT Gateway (if internet access is needed)
-   This means:
    -   Instances can reach the internet,
    -   But the internet cannot reach them — no inbound access.

### Why Do We Use Subnets?

-   Creating Security Layers
    -   A subnet is an isolation tool. Subnets behave like security boundaries.
Each subnet has its own Route Table and Security Group relationships.
        -   Public subnet: can communicate with the external world ➜ (e.g., web server)
        -   Private subnet: only accesses the internet internally via NAT, cannot be reached from outside ➜ (e.g., backend, database)

-   High Availability
    -   By distributing subnets across different Availability Zones, you prevent application downtime. Example:
        -   Public Subnet A (eu-central-1a)
        -   Public Subnet B (eu-central-1b)
    -   A Load Balancer sends traffic to both subnets.
    -   If one AZ fails, the other continues serving. Without subnets, you couldn’t make this separation.

-   Traffic Routing and Network Management
    -   Each subnet has its own dedicated Route Table. This allows:
        -   Public subnet to route traffic to the internet through an Internet Gateway.
        -   Private subnet to route traffic to the internet through a NAT Gateway.
        -   A Database subnet to accept only local traffic.

-   Performance and Organization
    -   Each application layer placed in its own subnet makes management easier.
    -   You define firewall rules more cleanly and logically.

## Internet Gateway (IGW), NAT Gateway, Route Table
### What is an Internet Gateway?

It is the door that connects your VPC to the internet. An Internet Gateway is attached directly to the VPC.

-   It is used only by Public subnets.
(Private subnets cannot use it since they do not have direct internet access.)

An Internet Gateway works bidirectionally; it manages outbound and inbound internet traffic.
-   Outbound Traffic: EC2 → IGW → Internet
    -   Route table: 0.0.0.0/0 → igw-xxxx
-   Inbound Traffic: Internet → IGW → EC2
    -   Route table: 10.0.0.0/16 → local

### What is a NAT Gateway?

A NAT Gateway works one-way — it does not accept incoming requests from the internet. It allows Private subnets to access the internet.
-   A NAT Gateway is created inside a Public Subnet (because it needs internet access).
-   It is used only by Private subnets.

-   Outbound traffic flow : Private EC2 → NAT Gateway → IGW → Internet
    -   Route table: 0.0.0.0/0 → nat-xxxx
    -   Important Note: A NAT Gateway does not accept new inbound connections from the internet. It only handles responses to connections initiated by the private instance itself. (e.g., response to a sudo apt update request)

### What is a Route Table?

A Route Table is the mechanism that controls all traffic routing decisions for a subnet.
It manages both:
-   Incoming traffic from the internet to the subnet
-   Outgoing traffic from the subnet to the internet

Each subnet uses one Route Table.

How It Works (Core Logic)
-   The fundamental logic works like this:
    -   Where is this IP going?
    -   Is it within the VPC? Internet? Another subnet?
    -   Which gateway should be used?
        -   Public subnet uses an Internet Gateway (IGW) to reach the internet: 0.0.0.0/0 → igw-xxxx
        -   Private subnet uses a NAT Gateway to reach the internet: 0.0.0.0/0 → nat-xxxx

**Default Route Table Rule**

Every Route Table created by AWS automatically includes one mandatory rule: 10.0.0.0/16 → local

This rule means:
-   This IP range belongs to the VPC.
-   Traffic flows locally between EC2 instances without using IGW or NAT.
-   All VPC internal communication is handled as LOCAL.
-   This rule cannot be deleted or modified.

### Traffic Flow Examples

#### 1. Public & Private EC2 Instances – Incoming Request from the Internet and Outgoing Response

Scenario : A user opens http://web.burak.com  in their browser. This request will reach the web server running on our EC2 instance. In this scenario, the traffic flow is identical for both Public and Private subnets.

-   **Incoming Traffic – Request**
    -   Summary : (Internet ➜ Elastic IP ➜ IGW ➜ Public Subnet ➜ Route Table ➜ ELB ➜ Security Group ➜ Target EC2 Web Server)

    -   DNS resolves the domain and sends the request to the Elastic IP (static IP).
    -   Traffic reaches the Internet Gateway (IGW).
    -   IGW is attached to the VPC, so it forwards the request to the public subnet.
    -   Public Subnet Route Table says:
        -   10.0.0.0/16 → local
        -   Meaning: If the IP is inside the VPC, forward it to the ELB.
    -   Security Group becomes active. If the incoming port is allowed, the request is forwarded to the EC2 web server in the ELB’s Target Group.


-   **Outgoing Traffic – Response**
    -   Summmary : (EC2 Web Server ➜ Security Group ➜ Route Table ➜ ELB ➜ Route Table ➜ IGW ➜ Internet)
    
    -   EC2 wants to send the response back to the internet.
    
    -   Security Group becomes active again. If outbound access is allowed, traffic proceeds to the Route Table.
    
    -   EC2 checks the Private Subnet Route Table (even if it’s a public subnet scenario, this logic remains consistent):
        -   10.0.0.0/16 → local
        -   Meaning: If the ELB is inside the VPC, send traffic locally.
    
    -   EC2’s response reaches the ELB first. (ELB Security Group automatically allows return traffic.)
    
    -   To send the response back to the user, the ELB must reach the internet, so the Public Subnet Route Table is used:   
        -   0.0.0.0/0 → igw-xxxx
        -   Meaning: Traffic destined for the internet goes out through the IGW.
    
    -   Finally, the response is sent back to the user through the IGW.


#### 2. Request from a Private EC2 Instance to the Internet

Scenario:  An EC2 instance inside a Private Subnet sends a request to the internet (e.g., sudo apt update).

-   **Outgoing Traffic – Request**

    -   Summary : (EC2 Web Server ➜ Security Group ➜ Route Table ➜ NAT Gateway ➜ IGW ➜ Internet)

    -   The EC2 instance wants to make an outbound request to the internet (e.g., sudo apt update).

    -   Security Group becomes active. If outbound rules allow it, the traffic proceeds to the Route Table.

    -   EC2 checks the Private Subnet Route Table. The Route Table says:
        -   0.0.0.0/0 → nat-xxxx
        -   Meaning: all outbound internet traffic must go to the NAT Gateway.

    -   Traffic reaches the NAT Gateway.

    -   NAT Gateway sends this traffic to the Internet Gateway (IGW).

    -   The request (e.g., to package repositories or update servers) reaches the external destination.

-   **Incoming Traffic – Response**

    -   Summary : (Internet ➜ IGW ➜ NAT Gateway ➜ Private Subnet Route Table ➜ EC2 Web Server)

    -   The external server (e.g., Ubuntu/Debian package repo) wants to send the response back.

    -   The response first reaches the IGW.

    -   IGW forwards this response to the NAT Gateway.

    -   NAT Gateway recognizes that this session was initiated by the Private EC2 instance → It sends the packets back to the EC2.

    -   NAT Gateway forwards the response to the Private Subnet Route Table, which routes it:
        -   10.0.0.0/16 → local
        -   Meaning: deliver the response locally to the Private EC2 instance.

    -   The EC2 instance receives the response and completes the request (e.g., package lists updated successfully).

## Security

### Security Group (SG) – Instance Level Firewall

**What is a Security Group?**

-   It is the dedicated security firewall attached to an EC2 instance.
-   It is attached to the instance, not to the subnet.
-   It answers the question:
    -   “Who can reach this EC2, and who can this EC2 communicate with?”
-   Works as allow-only. You cannot write DENY rules.
-   When you create a VPC, all inbound traffic is blocked, and only outbound traffic is allowed by default.
-   SGs are stateful, meaning:
    -   If you allow inbound traffic, the outbound response is automatically allowed.

**Example Configuration**
-   Inbound
    -   80/tcp → 0.0.0.0/0
    -   443/tcp → 0.0.0.0/0
    -   22/tcp → Your IP only
-   Outbound
    -   All traffic → 0.0.0.0/0 (default open)

### NACL (Network Access Control List)
**What is a NACL?**
-   It is the security firewall at the subnet boundary.
    -   Works at the subnet level, not the instance level.
-   Unlike SGs, NACLs allow both ALLOW and DENY rules.
-   Unlike SGs, NACLs are stateless, meaning:
    -   If you allow inbound traffic, you must also explicitly allow the outbound return traffic.
-   Each subnet uses only one NACL.
-   When you create a VPC, inbound and outbound rules are fully allowed by default.
-   NACLs act as a broader, outer-layer security filter compared to SGs.

**Rule Evaluation Order**
-   Rules are evaluated top to bottom based on increasing rule numbers.
-   The first matching rule is applied, and all others are ignored.

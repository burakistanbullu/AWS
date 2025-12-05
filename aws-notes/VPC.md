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

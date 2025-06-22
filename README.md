# Building and Securing a Virtual-Private-Cloud
In a world where cloud computing powers everything from financial institutions to entertainment giants, secure and scalable network design has become non-negotiable. This project showcases the creation and hardening of a customized Amazon VPC—a key foundational skill for anyone working in cloud architecture today. Unlike AWS's default VPC, which simplifies early experimentation, this project focuses on deliberate configuration choices that mirror real-world demands such as regulatory compliance, precise traffic segmentation, and tightly controlled access.

To begin, a custom VPC within the North Virginia (us-east-1) region was created with a 10.0.0.0/16 IPv4 CIDR block, including a public subnet (10.0.0.0/24) and a private subnet (10.0.1.0/24), each supporting up to 65,536 IPs. An Internet Gateway was attached to handle outbound internet access for public resources, while private subnets remained isolated behind a _NAT Gateway_. This separation reinforces core cloud design patterns by controlling which resources are exposed to the internet and which remain internal.

Routing decisions were central to this setup. For the public subnet, a custom route table ensured that all outbound traffic (0.0.0.0/0) was directed through the Internet Gateway, enabling external communication. _In contrast, private subnet traffic was routed through a NAT Gateway, enabling secure outbound access without making those resources publicly accessible_.

Security was layered at both the instance and subnet level. Security Groups acted as virtual firewalls, managing rules for specific resources. For instance, public web servers allowed HTTP and SSH access from any IP, whereas private servers accepted traffic only from trusted internal sources. At the subnet level, custom Network ACLs were introduced using a “deny by default” approach. Initially blocking all traffic, these ACLs required explicit rules for allowed traffic, offering broader control and better protection for sensitive workloads.

However, manually configuring subnets, route tables and security groups can be very time consuming. This time, the goal was to use the VPC resource map to visually represent the desired network. It provides a clear view of how components connect and interact, making it easier to design, manage and troubleshoot without digging through configurations.

The distinction between public and private subnets was intentionally maintained. Public subnets hosted EC2 instances requiring global access, such as web servers, while private subnets sheltered backend systems from direct exposure. To facilitate access and manage servers, SSH was enabled using encrypted key pairs in .pem format. This safeguarded login credentials and followed best practices for remote administration. 

Server deployment followed strict network policies. Public instances were associated with permissive security groups to support web traffic. Private servers operated within more restrictive security groups, and their communication was confined to specific internal IP ranges and VPC components. These networking and security decisions ensured that resources were both functional and resilient against common attack vectors.

In order to establish a direct connection between two VPC, a VPC peering connection was created. A peering connection lets VPCs and their resources route traffic between them using their private IP addresses. Without a peering connection, data transfers between VPCs would use resources' public address - meaning VPCs have to communicate over the public internet. The requester VPC initiates the peering request, while the accepter VPC must approve it. After approval, traffic can flow between the VPCs without using public internet. Once the peering connection is created, both VPCs need updated route tables to direct traffic between them. 

Testing the VPC peering connection involves connecting to EC2 instances in both VPCs and ensuring they can communicate with each other. Instead of using SSH key pair, EC2 Instance Connect was used instead. Tools like ping or SSH was used to test connectivity between the instances, verifying that the VPC peering connection is functioning and allowing traffic to route between the VPCs via their private IP addresses. This step ensures that the VPC peering connection is functioning, allowing traffic to route between the VPCs via their private IP addresses. EC2 Instance Connect plays a crucial role in facilitating these connections, especially when troubleshooting, as it provides an easy way to connect to your EC2 instances even without manually managing SSH keys. 



Overall, this project not only demonstrates a grasp of AWS networking fundamentals, but also reflects an understanding of real-world best practices in cloud security and infrastructure design. It delivers a blueprint for building scalable, secure VPCs tailored to the needs of modern businesses, all while reinforcing core cloud principles such as least privilege, segmentation, and traffic control.

Future Improvements
- While using a single private key to access both the private and public servers simplified setup in this scenario, it also introduces security risks. Anyone who obtains that key can gain access to all connected instances, which makes protecting the private key absolutely critical.

conlusion
This dual layer takes security to the next level as traffic must pass through multiple checks, which reduces the chances of unwanted access. 

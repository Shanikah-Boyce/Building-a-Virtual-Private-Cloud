

In an era where cloud computing powers everything from financial institutions to entertainment giants, **secure and scalable network design** has become non-negotiable. This project showcases the creation and hardening of a customized Amazon Virtual Private Cloud (VPC)—a foundational skill for anyone working in cloud architecture today. Unlike AWS's default VPC, which simplifies early experimentation, this project focuses on **deliberate configuration choices that mirror real-world demands** such as regulatory compliance, granular traffic segmentation, and tightly controlled access.


VPC 1 was deployed in the North Virginia (us-east-1) region using the IPv4 CIDR block 10.0.0.0/16. It included two subnets: a public subnet (10.0.0.0/24) and a private subnet (10.0.1.0/24). These subnets were used to group resources with similar access requirements.

To enable internet access for resources in the public subnet, the option to auto-assign public IPv4 addresses was enabled. This ensures that any EC2 instance launched in the public subnet automatically receives a public IP address, eliminating the need for manual assignment.

An Internet Gateway was attached to the VPC to allow outbound internet connectivity for the public subnet. In contrast, the private subnet was configured behind a NAT Gateway, providing internet access for internal resources while keeping them inaccessible from the public internet.

This architecture follows cloud best practices by clearly separating public-facing components from internal systems.

**Routing decisions** were central to this setup. A custom route table for the public subnet ensured outbound traffic (`0.0.0.0/0`) flowed through the Internet Gateway, while the private subnet used a separate route table directing traffic through the NAT Gateway. This allowed internal resources to access the internet securely without exposing them to inbound connections from external sources.


Security was implemented at both the instance and subnet levels. **Security Groups** acted as virtual firewalls, managing rules for specific resources. For instance, public-facing EC2 instances were permitted to receive HTTP and SSH traffic from any IP, while private instances only allowed traffic from trusted internal addresses. In addition, **Network ACLs** were employed using a “deny by default” strategy. Initially blocking all traffic, the ACLs were then populated with specific rules to allow only sanctioned communication, enhancing defense-in-depth.

Server deployment aligned with subnet responsibilities. **Public EC2 instances** served internet-facing applications, while **private EC2 instances** hosted backend workloads with no external exposure. SSH access was configured using **encrypted `.pem` key pairs** to ensure secure authentication in line with best practices for cloud-hosted servers.


To test **VPC peering**, which enables communication between instances in separate VPCs using private IPs without traversing the public internet, I used the **VPC Resource Map** to visually configure two VPCs (10.1.0.0/16 and 10.2.0.0/16). This helped clearly illustrate the relationships between subnets, gateways, and route tables, reducing configuration errors and easing troubleshooting. **NextWork-1-VPC** was designated as the requester and paired with **NextWork-2-VPC** (the accepter). While both were in the same AWS account, I noted that peering also works across accounts, allowing secure resource sharing without public exposure. After the peering request was accepted, I updated the route tables to support two-way traffic.

For EC2 setup, I skipped creating key pairs and instead used **EC2 Instance Connect**, which relies on AWS-managed credentials. To maintain consistent public accessibility, I assigned **Elastic IPs**, which prevent downtime during instance restarts by eliminating the need for DNS propagation delays. Connectivity between EC2 instances was successfully verified using `ping` and SSH through EC2 Instance Connect, confirming stable communication over the peering connection.


I used the aws configure command to set up AWS credentials, region, and output format, generating access keys for programmatic access. To allow my EC2 instance to securely interact with S3, I created a temporary access key for operations like listing buckets and uploading files.

Since EC2 communicates with S3 over the public internet by default, I added a Gateway Endpoint for S3 and updated the VPC route table to ensure traffic stayed within AWS’s private network. A restrictive bucket policy using the aws:sourceVpce condition limited access to requests through the endpoint. This policy blocked console access and confirmed enforcement. Once the endpoint was correctly routed, traffic flowed securely—demonstrating tight access control and enhanced security.


To monitor network activity, VPC Flow Logs were enabled on the public subnet to capture traffic data—specifically, ping communication between the peered instances. These logs were stored in Amazon CloudWatch within a dedicated log group for the VPC. Logging was configured at one-minute intervals and recorded both accepted and rejected traffic.

CloudWatch Log Insights was then used to analyze the captured flow logs, offering visibility into source and destination IP addresses, ports, bytes transferred, and traffic outcomes. Using the query "Top 10 byte transfers by source and destination IP addresses", I was able to identify which IP pairs transferred the most data. This insight was instrumental in diagnosing issues and optimizing network performance.


Overall, this project not only demonstrates a strong grasp of AWS networking fundamentals, but also reflects an understanding of real-world best practices in cloud security and infrastructure design. It delivers a blueprint for building scalable, secure VPCs tailored to the needs of modern businesses, all while reinforcing core cloud principles such as **least privilege, segmentation, and traffic control**. This project illustrates how combining network segmentation with private service access and layered policy controls can significantly harden a cloud environment. By requiring traffic to pass through multiple security gates—from security groups and ACLs to endpoint conditions and route controls—it reduces exposure to unauthorized access and reinforces a security-first architecture mindset.

---

### Future Improvements

While using a single private key to access both the private and public servers simplified setup in this scenario, it introduces security risks. If that key is compromised, an attacker could gain access to all connected resources. A more secure approach would involve:

- Using separate SSH keys** for different instances or roles.
* Preferably, leveraging **IAM roles and AWS Systems Manager Session Manager** for access management and auditing. This eliminates the need to manage SSH keys directly on instances, enhancing security and simplifying access control.
* However, this project emphasizes that in real-world scenarios, **attaching an IAM role to the EC2 instance is a more secure and scalable approach**.



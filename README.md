## Secure and Scalable Network Design in AWS

In an era where cloud computing powers everything from financial institutions to entertainment giants, **secure and scalable network design** has become non-negotiable. This project showcases the creation and hardening of a customized Amazon Virtual Private Cloud (VPC)—a foundational skill for anyone working in cloud architecture today. Unlike AWS's default VPC, which simplifies early experimentation, this project focuses on **deliberate configuration choices that mirror real-world demands** such as regulatory compliance, granular traffic segmentation, and tightly controlled access.

---

### VPC Architecture and Core Networking

The VPC was deployed in the **North Virginia (us-east-1) region** using a `10.0.0.0/16` IPv4 CIDR block. It included one **public subnet** (`10.0.0.0/24`) and one **private subnet** (`10.0.1.0/24`). An **Internet Gateway** was attached to facilitate outbound internet traffic for resources in the public subnet, while the private subnet remained isolated behind a **NAT Gateway**. This setup enforced a common best practice in cloud design by separating public-facing services from internal systems.

**Routing decisions** were central to this setup. A custom route table for the public subnet ensured outbound traffic (`0.0.0.0/0`) flowed through the Internet Gateway, while the private subnet used a separate route table directing traffic through the NAT Gateway. This allowed internal resources to access the internet securely without exposing them to inbound connections from external sources.

To simplify the deployment and management of network components, a **VPC resource map** was used to plan and visualize relationships between subnets, gateways, route tables, and instances. This visual approach reduced configuration errors and made it easier to troubleshoot issues.

---

### Layered Security Implementation

Security was implemented at both the instance and subnet levels. **Security Groups** acted as virtual firewalls, managing rules for specific resources. For instance, public-facing EC2 instances were permitted to receive HTTP and SSH traffic from any IP, while private instances only allowed traffic from trusted internal addresses. In addition, **Network ACLs** were employed using a “deny by default” strategy. Initially blocking all traffic, the ACLs were then populated with specific rules to allow only sanctioned communication, enhancing defense-in-depth.

Server deployment aligned with subnet responsibilities. **Public EC2 instances** served internet-facing applications, while **private EC2 instances** hosted backend workloads with no external exposure. SSH access was configured using **encrypted `.pem` key pairs** to ensure secure authentication in line with best practices for cloud-hosted servers.

---

### Secure S3 Communication with VPC Endpoints

To demonstrate secure S3 operations, a temporary access key was generated for tasks like listing buckets, viewing contents, and uploading a file. However, this project emphasizes that in real-world scenarios, **attaching an IAM role to the EC2 instance is a more secure and scalable approach**.

Since EC2 instances communicate with services like S3 over the public internet by default, a **Gateway Endpoint was created for S3**. Adding the endpoint's route to the VPC route table ensured that communication occurred over AWS’s private network. A **restrictive bucket policy** was applied, allowing access only through the endpoint by specifying the `aws:sourceVpce` condition. Once saved, this immediately blocked access from the AWS Management Console, validating the policy. After associating the endpoint with the correct route table, traffic flowed securely through the endpoint. This setup showcased **fine-grained access control and improved data security**.

---

### Private Connectivity with VPC Peering

To enable private communication between separate VPCs, a **VPC peering connection** was established. This allowed instances in each VPC to communicate using private IPs, eliminating the need to traverse the public internet. Once the peering request was accepted, route tables in both VPCs were updated to permit bi-directional traffic flow. Connectivity between EC2 instances was verified using **EC2 Instance Connect**, replacing the need for manual SSH key handling. Connectivity tests, such as `ping -c 5` and SSH commands, confirmed successful traffic flow through the peering link.

---

### Network Monitoring and Analysis

For monitoring, **VPC Flow Logs** were enabled to track network activity. Logs were stored in Amazon **CloudWatch** within a custom log group dedicated to the VPC. The logging interval was configured to one minute and included all accepted and rejected traffic. **CloudWatch Log Insights** was used to analyze the captured data, providing visibility into source and destination IPs, bytes transferred, ports, and traffic outcomes. This data supported diagnostics and performance tuning across the network.

---

### Conclusion

Overall, this project not only demonstrates a strong grasp of AWS networking fundamentals, but also reflects an understanding of real-world best practices in cloud security and infrastructure design. It delivers a blueprint for building scalable, secure VPCs tailored to the needs of modern businesses, all while reinforcing core cloud principles such as **least privilege, segmentation, and traffic control**. This project illustrates how combining network segmentation with private service access and layered policy controls can significantly harden a cloud environment. By requiring traffic to pass through multiple security gates—from security groups and ACLs to endpoint conditions and route controls—it reduces exposure to unauthorized access and reinforces a security-first architecture mindset.

---

### Future Improvements

While using a single private key to access both the private and public servers simplified setup in this scenario, it introduces security risks. If that key is compromised, an attacker could gain access to all connected resources. A more secure approach would involve:

- Using separate SSH keys** for different instances or roles.
* Preferably, leveraging **IAM roles and AWS Systems Manager Session Manager** for access management and auditing. This eliminates the need to manage SSH keys directly on instances, enhancing security and simplifying access control.



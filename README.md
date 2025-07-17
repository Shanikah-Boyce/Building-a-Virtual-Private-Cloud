# 🔐 Secure AWS VPC Architecture 
## 🌟 Project Summary
As organizations increasingly embrace cloud computing, the demand for secure, scalable infrastructure is more critical than ever. This project focused on designing a custom Amazon Virtual Private Cloud (VPC) architecture that combines robust security with cloud-native performance and agility. The aim was to support sensitive workloads by enforcing strict access controls, enabling secure communication, and ensuring long-term scalability.

The solution adopted a layered defense-in-depth approach, integrating security at every level of the network stack. Every design decision, from subnet segmentation to inter-VPC communication, was made to strike the right balance between security, operational simplicity, and future scalability.

### Key Architectural Design
The architecture was deployed in the AWS North Virginia (us-east-1) region using the CIDR block 10.1.0.0/16, named NovaGrid-1-VPC. It was logically divided into two subnets to separate public-facing and internal resources:
- Public Subnet (10.1.0.0/24): Designed for internet-facing workloads, where EC2 instances are automatically assigned IPv4 addresses and can route through the Internet Gateway for external connectivity.
- Private Subnet (10.1.1.0/24): Reserved for backend services, with no direct internet access, ensuring complete isolation from external exposure.

This separation not only enforced security boundaries but also aligned with the principle of least privilege, reducing the risk surface across the infrastructure.

## 🛡️ Network Security
### 🔒 Subnet-Level Protection
Network ACLs (NACLs) were implemented as the first layer of defense:
- Public Subnet: Allows unrestricted inbound/outbound traffic for external services.
- Private Subnet: Permits only ICMP IPv4 traffic originating from the public subnet, supporting diagnostics while maintaining isolation.
  
This stateless layer provided coarse traffic filtering and helped catch misrouted or unauthorized traffic early in the packet flow.

### 🔐 Instance-Level Protection
Security Groups (SGs) offered a second, stateful layer of control:
- Public EC2 instances allowed HTTP and SSH access to support web applications and remote admin workflows.
- Private EC2 instances accepted traffic only from instances in the public subnet’s security group, ensuring controlled internal access.

Combining SGs with NACLs created a layered model that reduced the likelihood of accidental exposure or overly permissive access rules.

## 🔄 VPC Peering
To enable secure communication across isolated environments, VPC peering was configured between NovaGrid-1-VPC (10.1.0.0/16) and NovaGrid-2-VPC (10.2.0.0/16). This allowed instances in both networks to communicate via private IPs, without routing traffic over the public internet.
<img width="919" height="583" alt="Screenshot 2025-04-24 150942" src="https://github.com/user-attachments/assets/59e3195c-07a4-4789-9f06-c9a18b92f54e" />

VPC peering was intentionally chosen over options like AWS Transit Gateway because it offered simpler configuration and lower cost for a two-VPC architecture. It met the performance and connectivity requirements without introducing unnecessary complexity.

NovaGrid-2-VPC included a public subnet (10.2.0.0/24) and a private subnet (10.2.1.0/24). To maintain consistent access control, its security groups and NACLs mirrored those of NovaGrid-1. This duplication ensured predictable, uniform security behavior across the network.

Routing tables in both VPCs were updated after peering to enable bidirectional internal traffic, facilitating seamless communication between services, critical for shared backend operations and service discovery.
<img width="664" height="438" alt="Screenshot 2025-04-24 152819" src="https://github.com/user-attachments/assets/11371ced-ae04-4e7f-b727-fe1ae9907e4a" />


## ☁️ Private S3 Access
For workloads in the private subnet, access to Amazon S3 was enabled using a Gateway Endpoint, eliminating the need for NAT Gateways or public IPs. This ensured all S3 traffic remained on the AWS backbone, avoiding exposure to the public internet.

A strict bucket policy using the aws:sourceVpce condition was applied to allow access only from the designated VPC endpoint. This effectively blocked any unintended access, even from AWS Management Console logins, reinforcing a zero-trust posture for data storage.

This design choice reduced both cost and complexity while increasing security, making it ideal for applications handling sensitive data.

## 📈 Monitoring and Visibility
VPC Flow Logs were enabled on the public subnet and streamed to Amazon CloudWatch. These logs captured accepted and rejected traffic, offering critical insights into network behavior at one-minute intervals.

Using CloudWatch Log Insights, queries were executed to identify the top 10 EC2 data transfers by byte size. This analysis confirmed that traffic patterns aligned with expected use cases and helped validate the architecture’s effectiveness.

Monitoring wasn’t treated as an afterthought—it was foundational to the design. Visibility into traffic patterns ensured easier troubleshooting, compliance reporting, and performance tuning.

## 🧠 Lessons Learned
This project highlighted the effectiveness of layered network security—combining stateless controls at the subnet level (NACLs) with stateful instance-level firewalls (SGs). Careful segmentation between public and private resources enforced clear trust boundaries.

Another takeaway was the importance of designing for visibility from the start. Having VPC Flow Logs and centralized monitoring made diagnostics faster and helped verify assumptions made during the planning stage.

## 🚀 Final Outcome & Key Insights
The result was a highly secure, scalable AWS VPC architecture with strong monitoring, isolation, and operational clarity:
- Clear Subnet Strategy: Isolated environments for public and private resources reduced risk and simplified access control.
- Secure S3 Integration: Gateway Endpoints and scoped bucket policies ensured private, auditable access to data.
- Resilient VPC Peering: Enabled secure internal communication without public exposure.
- Layered Security Controls: NACLs and SGs together minimized the impact of misconfigurations.
- Operational Transparency: Monitoring with CloudWatch and Flow Logs enabled real-time visibility into network behavior.
Each component was selected with a specific tradeoff in mind—cost, manageability, and control. The final architecture proved resilient, observable, and easy to scale.

## 🚀 Opportunities for Improvement
### 🔐 Restrict SSH Access
While public EC2 instances allowed SSH from all IPv4 addresses to support EC2 Instance Connect, this represents a potential attack vector. A more secure approach would restrict SSH to known IP ranges (e.g., office VPN or a bastion host).

To maintain flexibility while improving security, it’s recommended to identify the IP ranges used by EC2 Instance Connect and apply them to the security group. Alternatively, automating IP updates via AWS Lambda could provide a safer, dynamic solution.

### 👁️ Extend Internal Visibility
Currently, Flow Logs are enabled only on the public subnet. Enabling them on the private subnet would offer deeper insight into internal (east-west) traffic. This visibility could reveal unusual patterns, detect misrouted packets, and improve overall network hygiene.

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

## Connectivity Testing Between EC2 Instances
Network connectivity between Amazon EC2 instances was validated using ping and  curl commands from a public-facing EC2 instance. Successful ICMP echo replies confirmed that inbound traffic to the private subnet was permitted by the configured security groups and NACLs, demonstrating internal accessibility.

The curl command to https://learn.nextwork.org/projects/aws-hosta-website-on-s3 further verified outbound internet access from the public EC2 instance, confirming the correct setup of the internet gateway, route tables, and the assignment of a public or Elastic IP address.

<img width="601" height="567" alt="Screenshot 2025-07-01 132430" src="https://github.com/user-attachments/assets/8183bb45-be1b-4b6b-8759-90e0b2ac1a1a" />

<img width="935" height="503" alt="Screenshot 2025-04-23 172042" src="https://github.com/user-attachments/assets/0deafb35-6920-4ff7-ace5-5ef8751486d7" />


## 🔄 VPC Peering
To enable secure communication across isolated environments, VPC peering was configured between NovaGrid-1-VPC (10.1.0.0/16) and NovaGrid-2-VPC (10.2.0.0/16). This allowed instances in both networks to communicate via private IPs, without routing traffic over the public internet.

<img width="919" height="583" alt="Screenshot 2025-04-24 150942" src="https://github.com/user-attachments/assets/59e3195c-07a4-4789-9f06-c9a18b92f54e" />

VPC peering was intentionally chosen over options like AWS Transit Gateway because it offered simpler configuration and lower cost for a two-VPC architecture. It met the performance and connectivity requirements without introducing unnecessary complexity.

NovaGrid-2-VPC included a public subnet (10.2.0.0/24) and a private subnet (10.2.1.0/24). To maintain consistent access control, its security groups and NACLs mirrored those of NovaGrid-1. This duplication ensured predictable, uniform security behavior across the network.

Routing tables in both VPCs were updated after peering to enable bidirectional internal traffic, facilitating seamless communication between services, critical for shared backend operations and service discovery.
<img width="1129" height="454" alt="Screenshot 2025-07-01 131239" src="https://github.com/user-attachments/assets/7f25796c-e3d5-47a1-b6d3-c44a498e3c46" />

<img width="664" height="438" alt="Screenshot 2025-04-24 152819" src="https://github.com/user-attachments/assets/11371ced-ae04-4e7f-b727-fe1ae9907e4a" />


## ☁️ Private S3 Access
For workloads operating within a private subnet, Amazon S3 access was secured through a VPC Gateway Endpoint. This configuration removed the need for NAT Gateways or public IPs, ensuring that all traffic to S3 remained within AWS’s internal infrastructure and did not traverse the public internet.

<img width="960" height="584" alt="Screenshot 2025-04-25 152827" src="https://github.com/user-attachments/assets/7c1cf647-8220-4d00-a007-1e46d7d5fd9b" />

To enforce a zero-trust model, a strict S3 bucket policy using the aws:SourceVpce condition was implemented. This guaranteed that only requests from the approved VPC Endpoint were allowed, effectively blocking other access paths, including the AWS Management Console.

<img width="1204" height="570" alt="Screenshot 2025-04-25 153329" src="https://github.com/user-attachments/assets/3d23ca16-ec38-483f-95b7-0a3ccd090439" />

To test the effectiveness of the security controls, the VPC Endpoint Policy was briefly modified to deny all access. Since the bucket was only reachable via the gateway, this immediately disabled access across all interfaces: AWS CLI, SDKs, and the Console, for workloads in the private subnet. This confirmed that the configuration was properly enforced and could be quickly shut down when needed.

<img width="1329" height="465" alt="Screenshot 2025-04-25 155454" src="https://github.com/user-attachments/assets/737b325d-ff39-4514-875d-de845b8bb74d" />

<img width="1346" height="97" alt="Screenshot 2025-04-25 154031" src="https://github.com/user-attachments/assets/68a6a4b2-28c2-48ce-8e1d-4f3d6782f6bb" />

The final configuration lowered risk exposure, eliminated costs associated with NAT Gateways, and streamlined traffic handling. This made it an ideal solution for protecting sensitive workloads in environments that demand strict network control.

## 📈 Monitoring and Visibility
VPC Flow Logs were enabled on the public subnet and streamed to Amazon CloudWatch. These logs captured accepted and rejected traffic, offering critical insights into network behavior at one-minute intervals.

Using CloudWatch Log Insights, queries were executed to identify the top 10 data transfers by byte size. 
<img width="1328" height="528" alt="Screenshot 2025-07-02 123804" src="https://github.com/user-attachments/assets/3ee357e3-7793-4c54-a7cd-a202c8bd4e11" />

This analysis confirmed that traffic patterns aligned with expected use cases and helped validate the architecture’s effectiveness.
<img width="969" height="611" alt="Screenshot 2025-07-02 123836" src="https://github.com/user-attachments/assets/957774a4-466a-467e-9f0a-741c7add5f65" />


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

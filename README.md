# 🔐 Secure AWS VPC Architecture 
## 🌟 Project Summary
As organizations increasingly embrace cloud computing, the demand for secure, scalable infrastructure is more critical than ever. This project focuses on designing a custom Amazon Virtual Private Cloud (VPC) architecture that combines robust security with cloud-native performance and agility. The aim is to support sensitive workloads by enforcing strict access controls, enabling secure communication, and ensuring long-term scalability.

The solution implements a layered defense-in-depth approach, integrating security at every level of the network stack. Every design decision, from subnet segmentation to inter-VPC communication, strikes the right balance between security, operational simplicity, and future scalability.

# Key Architectural Design
The infrastructure is deployed in the AWS North Virginia (us-east-1) region with the CIDR block 10.1.0.0/16, named NovaGrid-1-VPC. To ensure effective resource segmentation, the VPC is logically divided into two subnets:

### VPC Configuration:
- Public Subnet (10.1.0.0/24): This subnet hosts internet-facing workloads, such as EC2 instances. These instances are automatically assigned public IPv4 addresses, with connectivity routed through the Internet Gateway for external access.

- Private Subnet (10.1.1.0/24): This subnet is dedicated to backend services and does not have direct internet access, ensuring full isolation from the outside world.

This design follows the principle of least privilege, minimizing the attack surface and reducing exposure to potential security risks.

### Network Security:
At the subnet level, Network Access Control Lists (NACLs) provide the first layer of defense:
- Public Subnet: Allows unrestricted inbound and outbound traffic for external services.
- Private Subnet: Only permits ICMP IPv4 traffic from the public subnet, enabling diagnostics while maintaining isolation.

Security Groups (SGs) offer stateful, granular control over traffic:
- Public EC2 instances: Allow only HTTP and SSH traffic to support web applications and remote administrator workflows.
- Private EC2 instances: Accept traffic only from the public subnet’s security group, ensuring controlled internal communication.

Combining SGs with NACLs provides a multi-layered security model, reducing the risk of unauthorized access.

# Connectivity Testing Between EC2 Instances
Network connectivity between the public and private EC2 instances was validated by running ping and curl commands from the public-facing EC2 instance. Successful ICMP echo replies confirmed that inbound traffic to the private instance was allowed by the configured security groups and NACLs, ensuring internal communication between the two instances was functioning correctly.

<img width="706" height="568" alt="image" src="https://github.com/user-attachments/assets/5b676fea-fe55-4799-91b7-00b05c76e1d0" />

To test outbound internet access, the curl command `curl https://learn.nextwork.org/projects/aws-host-a-website-on-s3` was executed from the public EC2 instance. This verified that the internet gateway, route tables, and assignment of a public IP or Elastic IP were properly configured, ensuring the public instance could access external resources.

<img width="857" height="547" alt="image" src="https://github.com/user-attachments/assets/0582fdf7-d887-4bc7-8a3c-46a7fa7b25a9" />

# 🔄 Inter-VPC Communication and Expansion
To accommodate growth and support additional workload isolation, a second VPC, NovaGrid-2, was provisioned with the CIDR block 10.2.0.0/16. Like its predecessor, it includes:
- Public Subnet (10.2.0.0/24): Designed for internet-facing resources.
- Private Subnet (10.2.1.0/24): Reserved for backend operations.

NovaGrid-2 inherits the security posture of NovaGrid-1, ensuring consistent network security controls across both environments.

To enable secure, low-latency communication between the two VPCs, a VPC Peering Connection was established. This allows public instances in NovaGrid-1 and NovaGrid-2 to communicate using their private IP addresses, ensuring that traffic between them remains isolated from the public internet. Although the public subnets can route traffic to the internet via an Internet Gateway, all internal communication, especially traffic traversing the peering link, remains private and protected.

With routing tables updated to support bidirectional traffic, the two VPCs can now facilitate cross-VPC service discovery and integration. This setup supports a more resilient, distributed architecture while simplifying service orchestration across both VPCs.

VPC Peering was chosen over AWS Transit Gateway due to its simplicity and cost-effectiveness for a smaller, two-VPC setup. While Transit Gateway provides more advanced features such as centralized routing, inter-region connectivity, and scalability for larger network topologies, it is better suited for more complex environments with multiple VPCs and regions.

<img width="919" height="583" alt="Screenshot 2025-04-24 150942" src="https://github.com/user-attachments/assets/59e3195c-07a4-4789-9f06-c9a18b92f54e" />



<img width="1129" height="454" alt="Screenshot 2025-07-01 131239" src="https://github.com/user-attachments/assets/7f25796c-e3d5-47a1-b6d3-c44a498e3c46" />

<img width="664" height="438" alt="Screenshot 2025-04-24 152819" src="https://github.com/user-attachments/assets/11371ced-ae04-4e7f-b727-fe1ae9907e4a" />


## ☁️ Private S3 Access
For workloads operating within a private subnet, Amazon S3 access is secured through a VPC Gateway Endpoint. This configuration eliminates the need for NAT Gateways or public IPs, ensuring that all traffic to S3 remains within AWS’s internal infrastructure and does not traverse the public internet.

<img width="960" height="584" alt="Screenshot 2025-04-25 152827" src="https://github.com/user-attachments/assets/7c1cf647-8220-4d00-a007-1e46d7d5fd9b" />

To enforce a zero-trust model, a strict S3 bucket policy using the aws:SourceVpce condition is applied. This guarantees that only requests from the approved VPC Endpoint are allowed, effectively blocking other access paths, including the AWS Management Console.

<img width="1204" height="570" alt="Screenshot 2025-04-25 153329" src="https://github.com/user-attachments/assets/3d23ca16-ec38-483f-95b7-0a3ccd090439" />

To test the effectiveness of the security controls, the VPC Endpoint Policy is temporarily modified to deny all access. Since the bucket is only reachable via the gateway, this immediately disables access across all interfaces—AWS CLI, SDKs, and the Console, for workloads in the private subnet. This confirms that the configuration is properly enforced and can be quickly shut down when needed.

<img width="1329" height="465" alt="Screenshot 2025-04-25 155454" src="https://github.com/user-attachments/assets/737b325d-ff39-4514-875d-de845b8bb74d" />

<img width="1346" height="97" alt="Screenshot 2025-04-25 154031" src="https://github.com/user-attachments/assets/68a6a4b2-28c2-48ce-8e1d-4f3d6782f6bb" />

The final configuration reduces risk exposure, eliminates costs associated with NAT Gateways, and streamlines traffic handling. This makes it an ideal solution for protecting sensitive workloads in environments that demand strict network control.

## 📈 Monitoring and Visibility
VPC Flow Logs are enabled on the public subnet and streamed to Amazon CloudWatch. These logs capture accepted and rejected traffic, offering critical insights into network behavior at one-minute intervals.

Using CloudWatch Log Insights, queries are run to identify the top 10 data transfers by byte size.
<img width="1328" height="528" alt="Screenshot 2025-07-02 123804" src="https://github.com/user-attachments/assets/3ee357e3-7793-4c54-a7cd-a202c8bd4e11" />

This analysis confirms that traffic patterns align with expected use cases and helps validate the architecture’s effectiveness.
<img width="2048" height="953" alt="image" src="https://github.com/user-attachments/assets/6435035b-0119-4aa7-9e7e-795d3c6302c8" />

Monitoring isn’t treated as an afterthought—it is foundational to the design. Visibility into traffic patterns enables easier troubleshooting, compliance reporting, and performance tuning.

## 🧠 Lessons Learned
This project demonstrates the effectiveness of layered network security—combining stateless controls at the subnet level (NACLs) with stateful instance-level firewalls (SGs). Careful segmentation between public and private resources establishes clear trust boundaries.

Another takeaway is the importance of designing for visibility from the start. Having VPC Flow Logs and centralized monitoring speeds up diagnostics and validates assumptions made during the planning stage.

## 🚀 Final Outcome & Key Insights
The result was a highly secure, scalable AWS VPC architecture with strong monitoring, isolation, and operational clarity:
- Defined Subnet Strategy: Isolated environments for public and private resources enhance security and simplify access control.
- Secure S3 Integration: Gateway Endpoints and scoped bucket policies enable private, auditable access to data.
- Resilient VPC Peering: Supports secure internal communication without public exposure.
- Layered Security Controls: NACLs and SGs together limit the impact of misconfigurations.
- Operational Transparency: Monitoring with CloudWatch and Flow Logs delivers real-time visibility into network behavior.


## 🚀 Opportunities for Improvement
### 🔐 Restrict SSH Access
To enable EC2 Instance Connect, public EC2 instances currently allow SSH access from any IPv4 address. This setup eliminates the need for manual key management, streamlining access for administrators. However, it also significantly increases the attack surface by exposing port 22 to unrestricted internet traffic. A more secure configuration involves restricting SSH access to trusted IP ranges, particularly those assigned to a bastion host. This helps prevent unauthorized connections and aligns with best practices in cloud security. Identifying the IP ranges used by EC2 Instance Connect and applying them within security group rules strengthens access control. In dynamic environments, automating these IP updates using AWS Lambda and Systems Manager provides a scalable, secure, and compliant solution with minimal manual intervention.

### 👁️ Extend Internal Visibility
Flow Logs are currently enabled only on the public subnet, limiting visibility into internal traffic. This hinders detection of internal anomalies or troubleshooting of access issues.

Extending Flow Logs to private subnets would offer deeper insights into internal activity—surfacing unusual traffic patterns, revealing misrouted packets, and improving network hygiene. This data can also be integrated into tools like CloudWatch Logs Insights or Amazon Athena for advanced monitoring and analytics, supporting more informed decision-making and enhancing operational oversight.

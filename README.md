# 🔐 Secure AWS VPC Architecture 
## 🌟 Project Summary
As organizations embrace the cloud, secure and scalable infrastructure becomes essential. This project showcases a tailored Amazon Virtual Private Cloud (VPC) architecture that fuses robust security with native cloud performance and elasticity. It is purpose-built to support sensitive workloads by enforcing access controls, securing communications, and ensuring long-term growth.

The architecture embraces a defense-in-depth strategy, embedding security across all network layers. Each design element, from subnet segmentation to inter-VPC connectivity, balances operational simplicity with hardened security and scalability.

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

<img src="https://github.com/user-attachments/assets/5b676fea-fe55-4799-91b7-00b05c76e1d0" width="700"/>

To test outbound internet access, the curl command `curl https://learn.nextwork.org/projects/aws-host-a-website-on-s3` was executed from the public EC2 instance. This verified that the internet gateway, route tables, and assignment of a public IP or Elastic IP were properly configured, ensuring the public instance could access external resources.

<img src="https://github.com/user-attachments/assets/0582fdf7-d887-4bc7-8a3c-46a7fa7b25a9" width="700"/>

# 🔄 Inter-VPC Communication and Expansion
<img src="https://github.com/user-attachments/assets/9ecad038-48f2-4efd-94e4-ac6f10a78cba"/>


As infrastructure needs evolved, NovaGrid expanded into a dual-VPC architecture with the provisioning of NovaGrid-2 (10.2.0.0/16). This second VPC was designed to support workload separation and long-term scalability while maintaining symmetry with NovaGrid-1. It includes a public subnet (10.2.0.0/24) for internet-facing resources and a private subnet (10.2.1.0/24) for backend services requiring isolation from external exposure.

<img src="https://github.com/user-attachments/assets/59e3195c-07a4-4789-9f06-c9a18b92f54e" width="700"/>

By distributing workloads across separate VPCs, NovaGrid reinforces both scalability and security. Production, staging, and development environments are now hosted in fully isolated zones, enabling each to scale independently without performance bleed. This separation also serves as a containment boundary, limiting the blast radius of configuration errors or security breaches and helping safeguard mission-critical data and services.

To enable secure, low-latency interconnectivity, a VPC Peering Connection was established between NovaGrid-1 and NovaGrid-2. This setup allows resources, especially those in public subnets, to exchange traffic using private IP addresses, keeping inter-VPC communication encrypted and invisible to the public internet. Bidirectional routing ensures both environments can access internal services reliably, strengthening integration while preserving isolation. 

<img src="https://github.com/user-attachments/assets/7f25796c-e3d5-47a1-b6d3-c44a498e3c46" width="700" />
<img src="https://github.com/user-attachments/assets/11371ced-ae04-4e7f-b727-fe1ae9907e4a" width="700" />

The decision to use VPC Peering over AWS Transit Gateway was deliberate as peering offers a direct, lightweight solution tailored for a two-VPC architecture. While Transit Gateway shines in complex, multi-VPC or cross-region ecosystems, it introduces additional overhead unnecessary for NovaGrid's current scale. Peering provides just the right balance of simplicity, performance, and cost-effectiveness, with architectural headroom for future growth.


## ☁️ Private S3 Access
For workloads operating within a private subnet, Amazon S3 access is secured through a VPC Gateway Endpoint. This configuration eliminates the need for NAT Gateways or public IPs, ensuring that all traffic to S3 remains within AWS’s internal infrastructure and does not traverse the public internet.


To enforce a zero-trust model, a strict S3 bucket policy using the aws:SourceVpce condition is applied. This guarantees that only requests from the approved VPC Endpoint are allowed, effectively blocking other access paths, including the AWS Management Console.

<img src="https://github.com/user-attachments/assets/7c1cf647-8220-4d00-a007-1e46d7d5fd9b" /> 
<img src="https://github.com/user-attachments/assets/3d23ca16-ec38-483f-95b7-0a3ccd090439" />

To test the effectiveness of the security controls, the VPC Endpoint Policy is temporarily modified to deny all access. Since the bucket is only reachable via the gateway, this immediately disables access across all interfaces, AWS CLI, SDKs, and the Console, for workloads in the private subnet. This confirms that the configuration is properly enforced and can be quickly shut down when needed.

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

# 🔐 Secure AWS VPC Architecture 
## 🌟 Project Summary
As organizations increasingly adopt cloud solutions, the need for secure and scalable infrastructure has never been greater. This project aimed to design a custom Amazon Virtual Private Cloud (VPC) architecture that balances robust security with cloud-native performance and agility. The goal was to build a resilient network tailored for sensitive workloads, enforcing strict access controls, enabling secure communication, and supporting long-term scalability. A layered defense-in-depth strategy formed the backbone of this solution, integrating security at every stage without compromising flexibility.

### Key Architectural Features
- Private Amazon S3 Access: Provides secure, internal-only access to S3 resources, preventing exposure to the public internet.
- Seamless Inter-VPC Connectivity: Facilitates secure communication between VPCs, ensuring the isolation of internal traffic from external networks.
- Enhanced Monitoring: Integrates Amazon CloudWatch and VPC Flow Logs for continuous monitoring, visibility, and insights into network traffic.

This architecture not only strengthens data confidentiality but also helps maintain regulatory compliance, empowering organizations to confidently scale their cloud footprint

## 🏗️ Architectural Design
To achieve the project’s objectives, the VPC was implemented in the AWS North Virginia (us-east-1) region, using the CIDR block 10.1.0.0/16, designated as NovaGrid-1-VPC. It was segmented into two subnets:
- Public Subnet (10.1.0.0/24): Designed for internet-facing workloads, where EC2 instances are automatically assigned IPv4 addresses and can route through the Internet Gateway for external connectivity.
- Private Subnet (10.1.1.0/24): Reserved for internal services, with no external routing, ensuring complete isolation from public internet exposure.

## 🛡️ Network Security
### 🔒 Subnet-Level Protection
Network ACLs (NACLs) provide stateless traffic control, acting as the first layer of defense:
- Public Subnet: Allows unrestricted inbound/outbound traffic for external services.
- Private Subnet: Permits only ICMP IPv4 traffic originating from the public subnet, supporting diagnostics while maintaining isolation.
### 🔐 Instance-Level Protection
Security Groups (SGs) serve as stateful firewalls, enabling fine-grained control over each resource:
- Public EC2 Instances: Permitted HTTP and SSH traffic from any IP address to support public-facing applications.
- Private EC2 Instances: Restricted to receive traffic solely from the NovaGrid Public SG, ensuring secure communication within the VPC.

## 🔄 VPC Peering
To securely connect resources across isolated VPC environments, VPC peering was configured between NovaGrid-1-VPC (10.1.0.0/16) and NovaGrid-2-VPC (10.2.0.0/16). This setup allowed instances in both VPCs to communicate using private IP addresses, completely bypassing the public internet.

### NovaGrid-2-VPC Configuration
For visual guidance, the AWS VPC Resource Map (a feature released in 2023) was used. NovaGrid-2-VPC was provisioned with the following:
- Public Subnet: 10.2.0.0/24
- Private Subnet: 10.2.1.0/24
To maintain consistent security policies across both VPCs, the security group and Network ACL rules from NovaGrid-1-VPC were mirrored in NovaGrid-2-VPC, ensuring uniform access control across both environments.

## Peering Details
- Requester: NovaGrid-1-VPC
- Accepter: NovaGrid-2-VPC
Though both VPCs reside in the same AWS account, the solution remains compatible with cross-account peering, enabling secure resource sharing beyond a single tenant.
Once the peering connection was accepted, route tables in both VPCs were updated to support bi-directional traffic, ensuring seamless internal communication between resources.

## ☁️ Private S3 Access
To improve security and prevent public internet exposure, the private subnet was configured to access Amazon S3 via a Gateway Endpoint, keeping all S3 traffic within AWS’s private network. This ensured all S3 traffic remained within AWS’s private network, enabling secure communication between EC2 instances and S3 without the need for public IPs or NAT gateways. A bucket policy was implemented to allow access only from the designated VPC endpoint, using the aws:sourceVpce condition. This blocked unauthorized access, including login attempts from the AWS Management Console, and reinforced strict security controls in line with a zero-trust model.

With routing validated and access restricted, all S3 traffic flowed securely through the endpoint, safeguarding sensitive data from exposure.

## 📈 Monitoring and Visibility
To maintain robust observability across the network, VPC Flow Logs were enabled on the public subnet and streamed directly to Amazon CloudWatch. These logs recorded both accepted and rejected traffic events at one-minute intervals, delivering high-resolution insights into network behavior.

## Traffic Analysis
Leveraging CloudWatch Log Insights, sample queries were executed to surface the top 10 EC2 data transfers by byte size. This analysis spotlighted key traffic routes and confirmed the operational integrity of inter-VPC communication.

The logs not only revealed traffic patterns but also supported proactive diagnostics, performance optimization, and compliance reporting, ensuring that all data flows remained aligned with the intended security and architectural goals.

## 🧠 Lessons Learned
This project emphasized the power of layered security in AWS, achieved by combining Network ACLs at the subnet level and Security Groups at the instance level. The clear segmentation between public and private resources reinforced architectural isolation and precise access control.

## 🚀 Final Outcome & Key Insights
This project delivered a secure, scalable, and highly available AWS VPC architecture with strong visibility, access control, and operational clarity:
- Clear VPC Design: Visual mapping of subnets, route tables, and gateways reduced setup errors and simplified troubleshooting.
- Private S3 Access: Gateway Endpoints and scoped bucket policies (aws:sourceVpce) ensured all S3 traffic stayed within the private network.
- Strong Network Security: Isolated subnets with NACLs and Security Groups protected public and private resources.
- Seamless VPC Peering: Enabled secure, private communication between VPCs.
- Real-Time Monitoring: VPC Flow Logs and CloudWatch Insights revealed traffic patterns and supported diagnostics without manual effort.

## 🚀 Opportunities for Improvement
While the existing architecture supports a wide range of flexible use cases, several targeted enhancements can significantly improve its security, scalability, and observability.

## 🔐 Restrict SSH Access
SSH access should never be exposed to the entire internet, as this opens the door to brute-force attacks, port scans, and other intrusion attempts. A more secure approach is to restrict SSH to known IP ranges, such as a trusted office network or a designated bastion host. This strategy ensures unrestricted access to public HTTP/HTTPS services while tightly controlling administrative entry points.

In this case, SSH was configured to accept all IPv4 ranges to support EC2 Instance Connect, which uses a dynamic pool of IP addresses. To enforce more precise access controls without interrupting legitimate administrative workflows, it’s recommended to research and identify the currently used IP ranges for EC2 Instance Connect and update security groups accordingly.

## 👁️ Improve Internal Visibility
Extending VPC Flow Logs to include the private subnet would enhance visibility into internal network traffic, helping identify anomalous behavior and optimize resource usage. This added layer of telemetry strengthens troubleshooting capabilities and supports overall operational monitoring.










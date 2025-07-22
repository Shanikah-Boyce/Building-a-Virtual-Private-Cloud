# Secure AWS VPC Architecture 
<img width="1850" height="3060" alt="Overall VPC (1)" src="https://github.com/user-attachments/assets/9610800b-d732-4473-bfb8-87d88cc5659f" />

## Project Summary

As organizations embrace the cloud, secure and scalable infrastructure becomes essential. This project showcases a custom Amazon Virtual Private Cloud (VPC) architecture that fuses robust security with native cloud performance and elasticity. It is purpose-built to support sensitive workloads by enforcing access controls, securing communications, and ensuring long-term growth.

The architecture embraces a defense-in-depth strategy, embedding security across all network layers. Each design element, from subnet segmentation to inter-VPC connectivity, balances operational simplicity with hardened security and scalability.

## Key Architectural Design
NovaGrid-1 was manually deployed in the AWS North Virginia region (us-east-1) using a CIDR block of 10.1.0.0/16. The entire network infrastructure, including the VPC, subnets, route tables, and Internet Gateway, was built step-by-step through the AWS Management Console. This hands-on approach ensured granular control over each configuration phase. 

To enhance resource segmentation and optimize traffic flow, the VPC has been logically divided into two subnets:

<img width="1870" height="1500" alt="NovaGrid-1 (1)" src="https://github.com/user-attachments/assets/92466770-06d3-4bd7-bb3b-6aacc84386a3" />


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

## Connectivity Testing Between EC2 Instances
Network connectivity between the public and private EC2 instances was validated by running ping and curl commands from the public-facing EC2 instance. Successful ICMP echo replies confirmed that inbound traffic to the private instance was allowed by the configured security groups and NACLs, ensuring internal communication between the two instances was functioning correctly.

<img src="https://github.com/user-attachments/assets/5b676fea-fe55-4799-91b7-00b05c76e1d0" width="700"/>

To test outbound internet access, the curl command `curl https://learn.nextwork.org/projects/aws-host-a-website-on-s3` was executed from the public EC2 instance. This verified that the internet gateway, route tables, and assignment of a public IP or Elastic IP were properly configured, ensuring the public instance could access external resources.

<img src="https://github.com/user-attachments/assets/0582fdf7-d887-4bc7-8a3c-46a7fa7b25a9" width="700"/>

## Inter-VPC Communication and Expansion
As NovaGrid’s infrastructure evolved, a dual-VPC architecture was adopted to meet growing operational demands. The introduction of NovaGrid-2, provisioned via the VPC Resource Map and assigned a distinct CIDR block (10.2.0.0/16), marked a strategic step toward improved scalability, enhanced network isolation, and greater architectural flexibility.
<img src="https://github.com/user-attachments/assets/9ecad038-48f2-4efd-94e4-ac6f10a78cba"/>

Designed to support workload separation and long-term scalability, NovaGrid-2 mirrors the structure of NovaGrid-1. It includes:
- A public subnet (10.2.0.0/24) for internet-facing resources
- A private subnet (10.2.1.0/24) for backend services requiring isolation from external access
<p align="center">
  <img width="1850" height="2600" alt="Peering (1)" src="https://github.com/user-attachments/assets/15463940-bdf7-405f-80a9-ba177355f4d8" />
</p>

Distributing workloads across separate VPCs has reinforced NovaGrid’s scalability and security posture. Production, staging, and development environments now reside in isolated zones, allowing independent scaling and minimizing the impact of misconfigurations or security incidents. This segmentation also creates strong blast-radius containment, protecting critical systems and data.

To enable secure, low-latency communication between environments, a VPC Peering Connection was established between NovaGrid-1 and NovaGrid-2.

<img src="https://github.com/user-attachments/assets/59e3195c-07a4-4789-9f06-c9a18b92f54e" width="700"/>

This setup allows resources, especially those in public subnets, to exchange traffic using their private IP addresses, keeping inter-VPC communication encrypted and invisible to the public internet. Bidirectional routing ensures both VPCs can reliably access internal services while preserving isolation. 

<img src="https://github.com/user-attachments/assets/7f25796c-e3d5-47a1-b6d3-c44a498e3c46" width="700" />
<img src="https://github.com/user-attachments/assets/11371ced-ae04-4e7f-b727-fe1ae9907e4a" width="700" />

The decision to use VPC Peering over AWS Transit Gateway was deliberate as peering offers a direct, lightweight solution tailored for a two-VPC architecture. While Transit Gateway shines in complex, multi-VPC or cross-region ecosystems, it introduces additional overhead unnecessary for NovaGrid's current scale. Peering provides just the right balance of simplicity, performance, and cost-effectiveness, with architectural headroom for future growth.

## ☁️ Private S3 Access
<img width="700" height="700" alt="S3" src="https://github.com/user-attachments/assets/28964f10-47e7-4c4a-be1c-cea8081cd50e" />

For workloads residing in a private subnet, access to Amazon S3 is securely facilitated via a VPC Gateway Endpoint. This setup removes the need for NAT Gateways or public IPs, ensuring that all S3 traffic stays within AWS’s internal network and avoids traversing the public internet.

To implement a zero-trust approach, a restrictive S3 bucket policy is applied using the aws:SourceVpce condition. 

<img src="https://github.com/user-attachments/assets/7c1cf647-8220-4d00-a007-1e46d7d5fd9b" /> 

This ensures that only requests originating from the designated VPC Endpoint are permitted, effectively blocking all other access paths, including those through the AWS Management Console.

<img src="https://github.com/user-attachments/assets/3d23ca16-ec38-483f-95b7-0a3ccd090439" />

As part of a security validation exercise, the VPC Endpoint policy was temporarily set to deny all access. Since the Amazon S3 bucket is only reachable via the Gateway Endpoint, this action immediately blocked all connectivity, through AWS CLI, SDKs, and the Management Console.

<img width="1329" height="465" alt="Screenshot 2025-04-25 155454" src="https://github.com/user-attachments/assets/737b325d-ff39-4514-875d-de845b8bb74d" />

<img width="1346" height="97" alt="Screenshot 2025-04-25 154031" src="https://github.com/user-attachments/assets/68a6a4b2-28c2-48ce-8e1d-4f3d6782f6bb" />

The goal was to confirm that if a threat actor attempted to misuse the endpoint, access could be swiftly and completely revoked. The outcome validated that the configuration enforces the zero-trust model as intended.

The final configuration minimizes risk exposure, eliminates NAT Gateway overhead, and simplifies traffic management, making it a robust solution for securing sensitive workloads in environments with stringent network control requirements.

## 📈 Monitoring and Visibility
<p align="center">
  <img width="700" height="700" alt="Blank diagram (1)" src="https://github.com/user-attachments/assets/0615da40-cb5f-43ae-bf0a-2f4e5cb34591" />
</p>
VPC Flow Logs are enabled on the public subnet and streamed to Amazon CloudWatch. These logs capture accepted and rejected traffic, offering critical insights into network behavior at one-minute intervals.

Using CloudWatch Log Insights, queries are run to identify the top 10 data transfers by byte size. This analysis confirms that traffic patterns align with expected use cases and helps validate the architecture’s effectiveness.
<p align="center">
  <img width="700" height="700" alt="Blank diagram (1)" src="https://github.com/user-attachments/assets/6435035b-0119-4aa7-9e7e-795d3c6302c8" />
</p>
Monitoring isn’t treated as an afterthought—it is foundational to the design. Visibility into traffic patterns enables easier troubleshooting, compliance reporting, and performance tuning.

## Conclusion
This project demonstrates the design and implementation of a secure, scalable AWS VPC architecture built on the principles of defense-in-depth, resource isolation, and proactive observability. Core architectural elements—such as structured subnet segmentation, private S3 access through Gateway Endpoints, scoped bucket policies, and resilient VPC peering—work together to safeguard sensitive workloads while maintaining cloud-native flexibility and performance.

The layered security model combines stateless Network ACLs with stateful Security Groups to reduce attack surface and enforce clear trust boundaries between public and private resources. VPC Flow Logs, integrated with Amazon CloudWatch, provide real-time visibility into network behavior. These logs were actively analyzed using CloudWatch Log Insights, which enabled effective validation of traffic patterns and rapid troubleshooting during testing phases.

For secure administrative access, EC2 Instance Connect was used in place of traditional SSH. This approach eliminates the need to open port 22 to the internet and ensures that remote access is managed through AWS-managed infrastructure. To further harden access controls, AWS’s published service IP ranges (ip-ranges.json) can be referenced to scope Security Group rules precisely to EC2 Instance Connect endpoints, reinforcing the zero-trust model.

Still, opportunities remain to enhance observability and reduce residual risks. Expanding VPC Flow Logs to private subnets would improve visibility into internal traffic and help uncover misconfigurations. The data collected can continue to be queried through CloudWatch Log Insights or Amazon Athena to deepen governance, anomaly detection, and performance analysis.

With these practices already in place and further refinements on the roadmap, the architecture is well-positioned to support sensitive cloud workloads with strong security, operational control, and long-term scalability.

\# AWS VPC Networking Project



\### Designing and Implementing a Secure AWS Network Infrastructure



A hands-on AWS networking project demonstrating how to build a custom Virtual Private Cloud (VPC), configure public and private subnets, establish internet connectivity, and control access using security groups and route tables.



<p align="center">

&#x20; <img src="architecture/architecture.png" alt="AWS VPC Networking Architecture" width="850">

</p>



<p align="center">

&#x20; <img src="https://img.shields.io/badge/AWS-Cloud-orange?logo=amazonaws\&logoColor=white" alt="AWS">

&#x20; <img src="https://img.shields.io/badge/Networking-VPC-blue" alt="VPC Networking">

&#x20; <img src="https://img.shields.io/badge/Compute-EC2-orange" alt="Amazon EC2">

&#x20; <img src="https://img.shields.io/badge/OS-Amazon%20Linux-232F3E" alt="Amazon Linux">

&#x20; <img src="https://img.shields.io/badge/Status-Completed-success" alt="Project Status">

</p>



\---



\## Overview



This project implements a custom AWS network environment with separate public and private subnets across two Availability Zones.



The infrastructure demonstrates public-facing web access, private-instance outbound connectivity through a NAT Gateway, and controlled inbound traffic using security groups.



\### Objectives



\- Create and configure a custom VPC.

\- Divide the network into public and private subnets.

\- Configure an Internet Gateway and NAT Gateway.

\- Implement public and private route tables.

\- Deploy public and private EC2 instances.

\- Apply security group rules following least-privilege principles.

\- Validate website access and outbound HTTPS connectivity.



\---



\## Architecture



The infrastructure was deployed in the \*\*Asia Pacific (Mumbai) Region — `ap-south-1`\*\*.



| Component | Configuration |

|---|---|

| VPC | `demo-vpc` |

| VPC CIDR | `10.0.0.0/16` |

| Public Subnet A | `10.0.1.0/24` |

| Public Subnet B | `10.0.2.0/24` |

| Private Subnet A | `10.0.11.0/24` |

| Private Subnet B | `10.0.12.0/24` |

| Internet Gateway | `demo-igw` |

| NAT Gateway | `demo-nat` |

| Public Route Table | `public-rt` |

| Private Route Table | `private-rt` |



\### Network Design



\*\*Public subnets\*\*

\- Associated with the public route table.

\- Use the Internet Gateway for internet-bound routes.

\- Host the public EC2 web server and NAT Gateway.



\*\*Private subnets\*\*

\- Associated with the private route table.

\- Route outbound internet traffic through the NAT Gateway.

\- Host the private EC2 instance without a public IPv4 address.



\*\*Availability Zones\*\*



The subnets span `ap-south-1a` and `ap-south-1b`, demonstrating subnet distribution across two Availability Zones.



\---



\## AWS Services Used



| Service | Purpose |

|---|---|

| Amazon VPC | Isolated virtual network |

| Amazon EC2 | Public web server and private instance |

| Internet Gateway | Internet connectivity for public resources |

| NAT Gateway | Outbound internet connectivity for private resources |

| Elastic IP | Public IP address associated with the NAT Gateway |

| Route Tables | Control network traffic routing |

| Security Groups | Control inbound and outbound instance traffic |

| AWS Systems Manager Session Manager | Access the private instance without a public IP |

| AWS IAM | Permissions for Systems Manager access |



\---



\## Implementation



\### 1. VPC and Subnet Configuration



Created a VPC with CIDR `10.0.0.0/16` and four subnets distributed across two Availability Zones.



\### 2. Internet Gateway and Public Routing



Attached an Internet Gateway to the VPC and configured the public route table with a default route:



`0.0.0.0/0 → Internet Gateway`



Associated both public subnets with this route table.



\### 3. NAT Gateway and Private Routing



Created a public NAT Gateway with an Elastic IP and configured the private route table with:



`0.0.0.0/0 → NAT Gateway`



This allows instances in the associated private subnets to initiate outbound internet connections without requiring public IPv4 addresses.



\### 4. EC2 Deployment



\- \*\*Public instance:\*\* Runs Amazon Linux 2023 and hosts an Apache web server.

\- \*\*Private instance:\*\* Runs Amazon Linux 2023 without a public IPv4 address.

\- \*\*Session Manager:\*\* Used to connect to the private instance through AWS Systems Manager.



\### 5. Security Groups



Configured security groups to control access to the instances.



\- Web security group: HTTP and HTTPS inbound access, with SSH restricted to an authorized source IP.

\- Private application security group: SSH access permitted from the web security group.

\- Database security group: Created a rule allowing TCP port `3306` from the web security group for demonstration purposes.



\*\*Note:\*\* A database server was not deployed as part of this project.



\---



\## Testing and Validation



| Test | Result |

|---|---|

| VPC and subnet configuration | Completed |

| Internet Gateway attachment | Completed |

| Public route table configuration | Completed |

| NAT Gateway provisioning | Completed |

| Private route table configuration | Completed |

| Public Apache website | Tested |

| Private EC2 access through Session Manager | Successful |

| Outbound HTTPS request from private EC2 | Successful |



\### Private Instance Connectivity Test



Executed an HTTPS request from the private EC2 instance:



```bash

curl -I --connect-timeout 10 https://amazon.com

```



The request returned an HTTP `301 Moved Permanently` response, redirecting to `https://www.amazon.com/`.



This verified that the private instance could resolve the domain and establish an outbound HTTPS connection through the configured network path.



\---



\## Project Screenshots



The screenshots below document the implementation and validation steps.



\### Network Infrastructure



| VPC Configuration | Subnets |

|---|---|

| !\[VPC Details](screenshots/01-vpc-details.png) | !\[Subnets](screenshots/02-subnets.png) |



| Internet Gateway | NAT Gateway |

|---|---|

| !\[Internet Gateway](screenshots/03-internet-gateway.png) | !\[NAT Gateway](screenshots/04-nat-gateway.png) |



\### Routing and Security



| Public Route Table | Private Route Table |

|---|---|

| !\[Public Route Table](screenshots/05-public-route-table.png) | !\[Private Route Table](screenshots/06-private-route-table.png) |



| Web Security Group | Private Security Group |

|---|---|

| !\[Web Security Group](screenshots/08-web-security-group-rules.png) | !\[Private Security Group](screenshots/10-private-security-group-rules.png) |



\### Deployment and Validation



| EC2 Instances | Working Website |

|---|---|

| !\[EC2 Instances](screenshots/07-ec2-instances.png) | !\[Website](screenshots/11-working-website.png) |



| Session Manager | Private HTTPS Test |

|---|---|

| !\[Session Manager](screenshots/12-session-manager.png) | !\[Private HTTPS Test](screenshots/13-private-https-test.png) |



\---



\## Key Learnings



Through this project, I gained practical experience with:



\- VPC design and IPv4 CIDR allocation.

\- Public versus private subnet architecture.

\- Route tables and internet routing.

\- Internet Gateway versus NAT Gateway functionality.

\- EC2 security groups and inbound traffic restrictions.

\- Private-instance administration using Systems Manager Session Manager.

\- Validating outbound connectivity from a private network.

\- Documenting cloud infrastructure with implementation screenshots.



\---



\## Potential Improvements



Possible extensions for a future iteration include:



\- Deploying an application behind an Application Load Balancer.

\- Adding a database in isolated private subnets.

\- Implementing VPC Flow Logs and CloudWatch monitoring.

\- Automating infrastructure provisioning with Terraform.

\- Evaluating a multi-AZ NAT architecture for higher availability.



These are potential improvements and are not part of the current deployment.



\---



\## Cost Considerations



AWS resources may incur charges, including NAT Gateway usage, EC2 instances, and public IPv4 addresses.



Stop or delete resources when they are no longer needed, after confirming that you have saved any required project evidence.



\---



\## Repository Structure



```text

aws-vpc-networking-project/

├── README.md

├── architecture/

│   └── architecture.png

└── screenshots/

&#x20;   ├── 01-vpc-details.png

&#x20;   ├── 02-subnets.png

&#x20;   ├── 03-internet-gateway.png

&#x20;   ├── 04-nat-gateway.png

&#x20;   ├── 05-public-route-table.png

&#x20;   ├── 06-private-route-table.png

&#x20;   ├── 07-ec2-instances.png

&#x20;   ├── 08-web-security-group-rules.png

&#x20;   ├── 09-database-security-group.png

&#x20;   ├── 10-private-security-group-rules.png

&#x20;   ├── 11-working-website.png

&#x20;   ├── 12-session-manager.png

&#x20;   └── 13-private-https-test.png

```



\---



<p align="center">

&#x20; Built as a hands-on learning project to explore AWS networking, routing, security, and private-instance connectivity.

</p>


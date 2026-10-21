\# AWS VPC Networking Project



\## Overview



This project demonstrates the design and configuration of a Virtual Private Cloud (VPC) on Amazon Web Services (AWS). It includes public and private subnets, internet connectivity, routing, security groups, and EC2 instances.



The project was built in the AWS Mumbai region (`ap-south-1`) to gain hands-on experience with AWS networking fundamentals.



\## Architecture



!\[AWS VPC Architecture](architecture.png)



\## AWS Services Used



\- Amazon VPC

\- Public and Private Subnets

\- Internet Gateway

\- NAT Gateway

\- Elastic IP

\- Route Tables

\- Amazon EC2

\- Security Groups

\- AWS Systems Manager Session Manager



\## VPC Configuration



| Property | Value |

|---|---|

| VPC Name | `demo-vpc` |

| CIDR Block | `10.0.0.0/16` |

| Region | Mumbai (`ap-south-1`) |

| DNS Resolution | Enabled |

| DNS Hostnames | Enabled |



\## Subnet Configuration



| Subnet | CIDR Block | Availability Zone | Type |

|---|---|---|---|

| public-subnet-a | `10.0.1.0/24` | ap-south-1a | Public |

| public-subnet-b | `10.0.2.0/24` | ap-south-1b | Public |

| private-subnet-a | `10.0.11.0/24` | ap-south-1a | Private |

| private-subnet-b | `10.0.12.0/24` | ap-south-1b | Private |



\## Network Architecture Components



\### Internet Gateway



The Internet Gateway (`demo-igw`) is attached to the VPC. The public route table directs internet-bound traffic through this gateway.



\### NAT Gateway



The NAT Gateway (`demo-nat`) is deployed in `public-subnet-a`. The private route table directs outbound internet traffic through the NAT Gateway, allowing private instances to access external services without requiring public IPv4 addresses.



\### Route Tables



\*\*Public Route Table — `public-rt`\*\*

\- Contains the local VPC route.

\- Routes `0.0.0.0/0` through the Internet Gateway.

\- Associated with both public subnets.



\*\*Private Route Table — `private-rt`\*\*

\- Contains the local VPC route.

\- Routes `0.0.0.0/0` through the NAT Gateway.

\- Associated with both private subnets.



\### EC2 Instances



\*\*Public Web Server — `public-web-server`\*\*

\- Runs Amazon Linux 2023.

\- Deployed in `public-subnet-a`.

\- Hosts a webpage using Apache HTTP Server.

\- The webpage was accessed successfully through its public IPv4 address.



\*\*Private Instance — `private-ec2-server`\*\*

\- Deployed in `private-subnet-a`.

\- Private IP: `10.0.11.214`.

\- Has no public IPv4 address.

\- Accessed using AWS Systems Manager Session Manager.



\### Security Groups



\- \*\*`web-sg`:\*\* Allows HTTP and HTTPS traffic. SSH access was restricted to the configured source IP during setup.

\- \*\*`private-app-sg`:\*\* Allows SSH traffic from `web-sg`.

\- \*\*`db-sg`:\*\* Configured with a TCP port 3306 rule from `web-sg`. No database was deployed in this project.



\## Testing and Validation



The following tests were performed:



\- Verified VPC and subnet configuration.

\- Verified Internet Gateway and NAT Gateway configuration.

\- Verified public and private route tables.

\- Accessed the webpage hosted on the public EC2 instance.

\- Connected to the private EC2 instance through Session Manager.

\- Tested outbound HTTPS connectivity from the private instance using:



```bash

curl -I --connect-timeout 10 https://amazon.com

```



The request returned an HTTP `301 Moved Permanently` response, confirming that the external HTTPS request received a response.



A failed ping test was not treated as conclusive evidence of an internet connectivity failure because ICMP traffic may be blocked.



\## Screenshots



The `screenshots/` directory contains AWS Console screenshots and test evidence for the VPC, subnets, gateways, route tables, EC2 instances, security groups, website, Session Manager, and HTTPS connectivity.



\## Key Learning Outcomes



\- VPC and subnet design.

\- IPv4 CIDR addressing.

\- Public and private network configuration.

\- Internet Gateway and NAT Gateway routing.

\- Route table configuration.

\- Security group access control.

\- EC2 and Apache web server deployment.

\- Private instance access using Session Manager.

\- Network connectivity testing and troubleshooting.



\## Potential Future Improvements



\- Automate infrastructure deployment with Terraform.

\- Configure VPC Flow Logs.

\- Review security group permissions using least privilege.

\- Add monitoring and alerting.



These are potential extensions, not completed features.



\## Repository Structure



```text

aws-vpc-networking-project/

├── README.md

├── architecture.png

└── screenshots/

&#x20;   ├── 01-vpc-details.png

&#x20;   ├── 02-subnets.png

&#x20;   ├── 03-internet-gateway.png

&#x20;   ├── 04-nat-gateway.png

&#x20;   ├── 05-public-route-table.png

&#x20;   ├── 06-private-route-table.png

&#x20;   ├── 07-ec2-instances.png

&#x20;   └── additional project screenshots

```



\## Cost Considerations



AWS resources may incur charges. NAT Gateways have hourly and data-processing costs, and EC2 instances and public IPv4 addresses may also incur charges.



Review your AWS resources and remove any you no longer need.



\---



\*\*Cloud Provider:\*\* Amazon Web Services  

\*\*Project Type:\*\* Hands-on Cloud Networking  

\*\*Region:\*\* Asia Pacific (Mumbai)


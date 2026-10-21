<div align="center">



\# AWS VPC Networking Project



\### Designing and Implementing Secure Network Infrastructure on AWS



A hands-on cloud networking project demonstrating custom VPC design, public and private subnet segmentation, internet routing, NAT Gateway connectivity, EC2 deployment, and security group configuration.



<br>



!\[AWS](https://img.shields.io/badge/Cloud-AWS-orange?logo=amazonaws\&logoColor=white)

!\[Amazon VPC](https://img.shields.io/badge/Networking-Amazon%20VPC-8C4FFF?logo=amazonaws\&logoColor=white)

!\[Amazon EC2](https://img.shields.io/badge/Compute-Amazon%20EC2-FF9900?logo=amazonec2\&logoColor=white)

!\[Linux](https://img.shields.io/badge/OS-Amazon%20Linux%202023-232F3E?logo=linux\&logoColor=white)

!\[Status](https://img.shields.io/badge/Project-Completed-success)



</div>



\---



\## Table of Contents



\- \[Project Overview](#-project-overview)

\- \[Architecture](#-architecture)

\- \[AWS Services](#-aws-services)

\- \[Network Configuration](#-network-configuration)

\- \[Implementation](#-implementation)

\- \[Security Configuration](#-security-configuration)

\- \[Testing and Validation](#-testing-and-validation)

\- \[Screenshots](#-screenshots)

\- \[Key Learnings](#-key-learnings)

\- \[Potential Improvements](#-potential-improvements)

\- \[Cost Considerations](#-cost-considerations)

\- \[Repository Structure](#-repository-structure)



\---



\## Project Overview



This project focuses on building a custom network infrastructure using \*\*Amazon Virtual Private Cloud (Amazon VPC)\*\*.



The objective was to understand how AWS networking components work together to provide controlled network access, public-facing web services, and outbound internet connectivity for private resources.



The infrastructure was configured in the \*\*Asia Pacific (Mumbai) Region (`ap-south-1`)\*\*, with four subnets distributed across two Availability Zones.



\### Project Objectives



\- Design a custom VPC with a structured IPv4 CIDR range.

\- Create public and private subnets across two Availability Zones.

\- Configure an Internet Gateway for public network connectivity.

\- Deploy a NAT Gateway for outbound connectivity from private subnets.

\- Configure public and private route tables.

\- Deploy public and private EC2 instances.

\- Apply security group rules to control network access.

\- Access the private EC2 instance using AWS Systems Manager Session Manager.

\- Validate website availability and outbound HTTPS connectivity.



\---



\## Architecture



The architecture separates public-facing resources from private resources while using route tables and gateways to control network traffic.



<div align="center">



<img src="architecture/architecture.png" alt="AWS VPC Networking Architecture" width="100%">



\*\*Figure 1: AWS VPC Networking Architecture\*\*



</div>



\### Architecture Components



| Component | Responsibility |

|---|---|

| Amazon VPC | Provides an isolated virtual network |

| Public Subnets | Host resources requiring public network connectivity |

| Private Subnets | Host resources without direct public IPv4 addresses |

| Internet Gateway | Enables internet routing for appropriately configured public resources |

| NAT Gateway | Allows private instances to initiate outbound internet connections |

| Public Route Table | Routes internet-bound traffic


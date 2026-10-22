# AWS VPC Networking: Public and Private Infrastructure

A hands-on AWS networking project demonstrating custom VPC design, public and private subnet segmentation, Internet Gateway and NAT Gateway routing, EC2 web hosting, security groups, private-instance administration with AWS Systems Manager Session Manager, and connectivity troubleshooting.

<p align="center">
  <img src="architecture/architecture.png" alt="AWS VPC Networking Architecture" width="850" />
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Cloud-AWS-orange?logo=amazonaws" alt="AWS" />
  <img src="https://img.shields.io/badge/Region-ap--south--1-blue" alt="AWS Mumbai Region" />
  <img src="https://img.shields.io/badge/Compute-Amazon%20EC2-FF9900?logo=amazonec2" alt="Amazon EC2" />
  <img src="https://img.shields.io/badge/Networking-VPC%20%7C%20IGW%20%7C%20NAT-232F3E?logo=amazonaws" alt="AWS Networking" />
  <img src="https://img.shields.io/badge/Access-Systems%20Manager-527FFF?logo=amazonaws" alt="AWS Systems Manager" />
  <img src="https://img.shields.io/badge/Status-Completed-success" alt="Project completed" />
</p>

---

## Project Overview

This repository documents the design, deployment, testing, and cleanup of a custom Amazon Virtual Private Cloud (VPC) in the AWS Mumbai Region (`ap-south-1`). The environment separated internet-facing infrastructure from private workloads using subnet design, route tables, and security groups.

The public tier contained an Amazon EC2 instance running Apache HTTP Server and serving a custom web page. The private tier contained an EC2 instance without a public IPv4 address. AWS Systems Manager Session Manager was used to access the private instance, and an outbound HTTPS request was tested through the NAT Gateway.

The project was created as a hands-on learning environment. After the configuration and connectivity tests, the AWS infrastructure was deleted to reduce the chance of continued resource charges. The local project files, screenshots, architecture diagram, and GitHub repository remain separate from the AWS resources.

## Key Objectives

- Design a custom VPC using a private IPv4 CIDR block.
- Create public and private subnets in two Availability Zones.
- Configure an Internet Gateway and public routing.
- Configure a NAT Gateway for private-subnet outbound internet access.
- Deploy and test an Apache web server on Amazon Linux 2023.
- Access a private EC2 instance using Systems Manager Session Manager.
- Apply security group rules to control network access.
- Verify connectivity and troubleshoot differences between ICMP and HTTPS tests.
- Document the implementation, test results, limitations, and cost-cleanup steps.

> **Project scope:** This project focuses on foundational VPC networking and EC2 connectivity. A database security group was configured for learning purposes, but no database or Amazon RDS instance was deployed. Terraform, a load balancer, and comprehensive VPC Flow Logs/CloudWatch monitoring were not implemented or validated.

## Architecture

### 1. VPC network and web tier

```text
                         INTERNET
                            |
                    Internet Gateway
                         demo-igw
                            |
             +--------------+--------------+
             |                             |
        PUBLIC SUBNET A               PUBLIC SUBNET B
        10.0.1.0/24                   10.0.2.0/24
        ap-south-1a                   ap-south-1b
             |                             |
      public-web-server              Reserved for future
      Amazon Linux 2023               public workloads
      Apache HTTP Server
             |
         demo-nat
      (NAT Gateway in
       public-subnet-a)
             |
             +------------------------------+
                            |
                  PRIVATE ROUTE TABLE
                            |
             +--------------+--------------+
             |                             |
      PRIVATE SUBNET A              PRIVATE SUBNET B
      10.0.11.0/24                  10.0.12.0/24
      ap-south-1a                   ap-south-1b
             |
      private-ec2-server
      No public IPv4 address
             |
      AWS Systems Manager
      Session Manager
```

The diagram is a logical overview rather than a complete traffic-flow drawing. The NAT Gateway was deployed in `public-subnet-a`. Both private subnets used the private route table, whose default IPv4 route targeted that NAT Gateway. No second NAT Gateway or automatic failover was configured.

### 2. Public traffic flow

1. A client requests the public web server's address.
2. The public subnet's route table directs internet-bound IPv4 traffic to the Internet Gateway.
3. The instance's security group permits configured HTTP traffic on TCP port 80.
4. Apache returns the custom web page.

The web security group also included an inbound TCP port 443 rule, but the project did not establish a separate Apache TLS configuration. The website access test was documented as a working website test, not as proof of a configured HTTPS origin on the EC2 instance.

### 3. Private outbound traffic flow

1. The private EC2 instance initiates an outbound connection.
2. Its subnet uses `private-rt`.
3. The `0.0.0.0/0` route forwards the traffic to `demo-nat`.
4. The NAT Gateway provides the outbound internet path through the public subnet and Internet Gateway.
5. Return traffic is delivered for the established connection.

A NAT Gateway supports outbound connections from private resources; it does not ordinarily permit unsolicited inbound internet connections to those instances.

### 4. Private instance administration

The private instance was managed with AWS Systems Manager Session Manager. This avoided the need to assign it a public IPv4 address or expose an inbound SSH endpoint directly to the internet. Successful access also depended on correct IAM permissions, the SSM agent, and connectivity to Systems Manager endpoints.

## AWS Resources

| Component | Resource name | Configuration or purpose |
|---|---|---|
| VPC | `demo-vpc` | Custom virtual network, `10.0.0.0/16` |
| Public subnet A | `public-subnet-a` | `10.0.1.0/24`, `ap-south-1a` |
| Public subnet B | `public-subnet-b` | `10.0.2.0/24`, `ap-south-1b` |
| Private subnet A | `private-subnet-a` | `10.0.11.0/24`, `ap-south-1a` |
| Private subnet B | `private-subnet-b` | `10.0.12.0/24`, `ap-south-1b` |
| Internet Gateway | `demo-igw` | Internet connectivity for public routing |
| NAT Gateway | `demo-nat` | Outbound internet path for private-subnet IPv4 traffic |
| Public route table | `public-rt` | Local VPC route and default route to Internet Gateway |
| Private route table | `private-rt` | Local VPC route and default route to NAT Gateway |
| Public EC2 instance | `public-web-server` | Amazon Linux 2023 and Apache web server |
| Private EC2 instance | `private-ec2-server` | Amazon Linux 2023, no public IPv4 address |
| Web security group | `web-sg` | HTTP/HTTPS inbound rules and SSH restricted to a source IP |
| Private app security group | `private-app-sg` | SSH ingress from `web-sg` |
| Database security group | `db-sg` | TCP port 3306 allowed from `web-sg`; no database deployed |
| IAM role | `EC2-SSM-Role` | Instance role with Systems Manager and CloudWatch Agent managed policies attached |

Resource names are specific to this learning environment. Both EC2 instances, the NAT Gateway, its Elastic IP, the VPC, and the associated networking resources were subsequently deleted during cleanup. The inventory describes the deployed configuration, not currently running infrastructure.

## Network Design

### VPC configuration

| Property | Value |
|---|---|
| VPC name | `demo-vpc` |
| IPv4 CIDR | `10.0.0.0/16` |
| Region | `ap-south-1` (Asia Pacific — Mumbai) |
| DNS resolution | Enabled |
| DNS hostnames | Enabled |

### Subnet configuration

| Subnet | CIDR block | Availability Zone | Intended role |
|---|---|---|---|
| `public-subnet-a` | `10.0.1.0/24` | `ap-south-1a` | Public web server and NAT Gateway |
| `public-subnet-b` | `10.0.2.0/24` | `ap-south-1b` | Additional public-subnet capacity |
| `private-subnet-a` | `10.0.11.0/24` | `ap-south-1a` | Private EC2 workload |
| `private-subnet-b` | `10.0.12.0/24` | `ap-south-1b` | Additional private-subnet capacity |

Each `/24` subnet contains 256 IPv4 addresses in its CIDR range. AWS reserves five addresses in each IPv4 subnet, leaving 251 addresses available for customer use before other service-specific constraints.

### Route table configuration

**Public route table — `public-rt`**

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Communication within the VPC |
| `0.0.0.0/0` | `demo-igw` | Internet-bound IPv4 traffic |

The public route table was associated with both public subnets.

**Private route table — `private-rt`**

| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | Local | Communication within the VPC |
| `0.0.0.0/0` | `demo-nat` | Outbound internet-bound IPv4 traffic |

The private route table was associated with both private subnets. A route to a NAT Gateway does not by itself make a subnet public; subnet classification depends on its route to an Internet Gateway and the full network configuration.

## Implementation

### 1. Create the VPC and subnets

- Created `demo-vpc` with CIDR `10.0.0.0/16`.
- Enabled DNS resolution and DNS hostnames.
- Created four `/24` subnets across `ap-south-1a` and `ap-south-1b`.
- Separated public-facing and private workloads into distinct subnet ranges.

### 2. Configure public routing

- Created `demo-igw` and attached it to the VPC.
- Created `public-rt`.
- Added the local VPC route and a default IPv4 route to the Internet Gateway.
- Associated the route table with both public subnets.

### 3. Configure private egress

- Created `demo-nat` in `public-subnet-a` with public connectivity.
- Allocated an Elastic IP for the NAT Gateway.
- Created `private-rt` with a default route to the NAT Gateway.
- Associated the private route table with both private subnets.

### 4. Configure security groups

Created `web-sg`, `private-app-sg`, and `db-sg` to demonstrate web access, restricted administration, and a possible database-tier access rule. `db-sg` was only a security configuration exercise; no database server was deployed.

### 5. Deploy the public web server

- Launched `public-web-server` using Amazon Linux 2023 in `public-subnet-a`.
- Installed Apache HTTP Server.
- Enabled and started the Apache service.
- Created a custom page at `/var/www/html/index.html`.
- Confirmed that the website was accessible through the public endpoint.

### 6. Deploy and access the private instance

- Launched `private-ec2-server` using Amazon Linux 2023 in `private-subnet-a`.
- Confirmed that it had no public IPv4 address.
- Attached `EC2-SSM-Role` with `AmazonSSMManagedInstanceCore` and `CloudWatchAgentServerPolicy`.
- Used AWS Systems Manager Session Manager to access the instance.

The CloudWatch Agent policy was attached to the role, but full agent installation and telemetry collection were not validated as part of the recorded tests.

### 7. Test private outbound HTTPS

Executed the following command from the private instance:

```bash
curl -I --connect-timeout 10 https://amazon.com
```

The request returned an HTTP `301 Moved Permanently` response with a redirect to `https://www.amazon.com/`. This demonstrated successful DNS resolution and an outbound HTTPS request for that test. The redirect was an application-level response, not a network failure.

### 8. Document and clean up

- Captured screenshots of the VPC, subnets, gateways, route tables, EC2 instances, security group rules, website, Session Manager, and HTTPS test.
- Added the architecture diagram and project documentation to GitHub.
- Documented troubleshooting, limitations, and cost considerations.
- Deleted the project infrastructure after testing to reduce ongoing costs.

## Security Design

### Security group rules

| Security group | Inbound rule | Source | Intended use |
|---|---|---|---|
| `web-sg` | TCP 80 | `0.0.0.0/0` | Public HTTP website traffic |
| `web-sg` | TCP 443 | `0.0.0.0/0` | HTTPS traffic rule; a separate TLS origin configuration was not validated |
| `web-sg` | TCP 22 | Restricted source IP | Administrative SSH access |
| `private-app-sg` | TCP 22 | `web-sg` | SSH from instances associated with the web security group, if routed and otherwise permitted |
| `db-sg` | TCP 3306 | `web-sg` | Example MySQL access rule; no database deployed |

Security groups are stateful virtual firewalls associated with network interfaces. Referencing another security group as a source can help limit access to workloads that use that group, but it should be designed around actual application roles rather than names alone.

### Security principles demonstrated

- **Network segmentation:** public and private workloads were placed in separate subnets.
- **Controlled inbound access:** the SSH rule on the web security group used a restricted source IP rather than unrestricted internet access.
- **Security group references:** the private application security group used `web-sg` as its SSH source.
- **Private administration:** Session Manager provided access without a public IPv4 address on the private instance.
- **Least-privilege awareness:** rules were constrained by protocol, port, and source where configured.

### Potential security improvements

- Prefer Session Manager over public SSH access when practical.
- Restrict administrative access and remove unused inbound rules.
- Configure valid TLS for any website that is intended to serve HTTPS directly from the instance.
- Enable and review VPC Flow Logs and suitable monitoring before production use.
- Review instance role policies and remove permissions that are not needed.
- Consider VPC endpoints for supported AWS services to reduce dependence on NAT egress.

## Verification and Test Results

| Test | Expected result | Recorded result |
|---|---|---|
| VPC and subnet design | CIDR ranges and Availability Zones match the plan | Configured |
| Public route table | Default route targets Internet Gateway | Configured |
| Private route table | Default route targets NAT Gateway | Configured |
| Apache service | Web server starts and serves custom content | Verified |
| Public website | Custom page accessible through public endpoint | Verified |
| Private EC2 addressing | No public IPv4 address | Verified |
| Session Manager | Private instance can be managed through SSM | Verified |
| Private outbound HTTPS | HTTPS request succeeds through the configured egress path | HTTP 301 response received |
| ICMP ping | Depends on ICMP and destination/network rules | 100% packet loss was observed; this did not establish that HTTPS egress was blocked |
| Infrastructure cleanup | Project resources are removed | Cleanup performed; review the AWS Billing console for final charges and any remaining resources |

These results describe the recorded project tests. They do not represent a formal penetration test, load test, or production-readiness assessment.

## Troubleshooting and Lessons Learned

### 1. Git repository and remote configuration

The repository setup required attention to the current working directory and remote URL.

**Lesson:** Confirm the directory and Git remote before committing or pushing:

```powershell
git status
git remote -v
```

Replace any placeholder remote with the actual GitHub repository URL.

### 2. Ping failure is not proof of internet failure

The private instance's ping test showed packet loss, while the HTTPS request returned a valid HTTP redirect.

**Lesson:** ICMP and HTTPS use different protocols and may be treated differently by network rules or destinations. A failed ping alone does not prove that outbound HTTPS is blocked.

Useful checks include:

```bash
getent hosts amazon.com
curl -I --connect-timeout 10 https://amazon.com
```

### 3. Public addressing and routing are separate concerns

An Internet Gateway alone does not make every resource publicly reachable. Routing, public IPv4 addressing, security group rules, and service configuration all matter.

**Lesson:** When troubleshooting access, check the effective route table, instance addressing, security group rules, and whether the application is listening on the expected port.

### 4. Private egress depends on a chain of components

Outbound HTTPS from the private instance depended on the private route table, NAT Gateway, public-subnet route to the Internet Gateway, and appropriate security controls.

**Lesson:** Trace the traffic path end to end before changing rules randomly.

### 5. IAM permissions do not replace networking

Session Manager relies on IAM permissions, the SSM agent, and connectivity to the Systems Manager service.

**Lesson:** IAM authorization and network reachability are different requirements. Both must be correct for management access to work.

### 6. A security group is not a deployed database

`db-sg` contained a database-oriented rule, but no database or RDS instance was created.

**Lesson:** In project documentation and interviews, distinguish resources that were configured from services that were actually deployed and tested.

### 7. Resource cleanup is part of cloud engineering

NAT Gateways, EC2 runtime, EBS volumes, public IPv4 addresses, snapshots, and data transfer can incur costs.

**Lesson:** Plan teardown after testing, verify resource deletion, and review billing. Stopping an EC2 instance does not necessarily remove storage or NAT Gateway charges.

## Cost Management and Cleanup

The project was a learning environment rather than a continuously running production service. Potential billable components included:

- NAT Gateway hourly usage and data processing.
- EC2 instance runtime.
- EBS volumes and snapshots.
- Public IPv4 addresses and Elastic IP usage.
- Data transfer and other services if enabled.

After the tests, the project infrastructure was deleted, including the EC2 instances, NAT Gateway, Elastic IP, custom subnets, route tables, security groups, Internet Gateway, and VPC. The IAM role was identified for review separately because deleting a VPC does not remove IAM roles.

**Billing note:** Deleting resources does not erase charges already incurred. AWS billing details may take time to update. Check AWS Billing and Cost Management and inspect relevant regions for any remaining resources before assuming that the final cost is zero.

## Project Limitations

This project demonstrates foundational AWS networking. The following capabilities were not implemented or not validated:

- No Amazon RDS or other database instance was deployed.
- No Application Load Balancer or Network Load Balancer was configured.
- No second application server or automatic failover was tested.
- No second NAT Gateway was created for Availability Zone-level egress redundancy.
- Infrastructure deployment with Terraform was not completed.
- Comprehensive VPC Flow Logs and CloudWatch Agent telemetry were not validated.
- No formal penetration test or production-scale load test was performed.
- The resources were deleted after the documented tests and are not intended to be a live demo environment.

These boundaries keep the project description aligned with the actual implementation and observed results.

## Future Improvements

### Networking and availability

- Configure a redundant NAT design across Availability Zones.
- Deploy multiple application instances across Availability Zones.
- Add an Application Load Balancer and test health checks and failover.
- Add VPC endpoints for supported AWS services where appropriate.
- Test failure scenarios and document expected traffic paths.

### Security and observability

- Enable VPC Flow Logs with an appropriate destination and retention policy.
- Configure CloudWatch dashboards, metrics, and alarms.
- Review and narrow IAM policies.
- Prefer Systems Manager Session Manager for administrative access.
- Add TLS configuration if HTTPS is served directly by the web server.

### Infrastructure as Code

- Rebuild the architecture using Terraform.
- Separate provider configuration, variables, outputs, and resources.
- Use plans and state management carefully.
- Add documented deployment, validation, and teardown procedures.

## Suggested Repository Structure

```text
aws-vpc-networking-project/
├── README.md
├── architecture/
│   └── architecture.png
└── screenshots/
    ├── 01-vpc-details.png
    ├── 02-subnets.png
    ├── 03-internet-gateway.png
    ├── 04-nat-gateway.png
    ├── 05-public-route-table.png
    ├── 06-private-route-table.png
    ├── 07-ec2-instances.png
    ├── 08-web-security-group-rules.png
    ├── 09-database-security-group.png
    ├── 10-private-security-group-rules.png
    ├── 11-working-website.png
    ├── 12-session-manager.png
    └── 13-private-https-test.png
```

This is the intended documentation layout based on the project files previously recorded. Confirm the actual local filenames before relying on the tree as an exact inventory. Remove non-project operating-system metadata such as `desktop.ini` if present.

## Screenshots

The screenshots document the configuration and tests carried out during the project. GitHub renders the relative links below when the files exist at these paths.

| Evidence | Screenshot |
|---|---|
| VPC configuration | [`01-vpc-details.png`](screenshots/01-vpc-details.png) |
| Subnet layout | [`02-subnets.png`](screenshots/02-subnets.png) |
| Internet Gateway | [`03-internet-gateway.png`](screenshots/03-internet-gateway.png) |
| NAT Gateway | [`04-nat-gateway.png`](screenshots/04-nat-gateway.png) |
| Public routes | [`05-public-route-table.png`](screenshots/05-public-route-table.png) |
| Private routes | [`06-private-route-table.png`](screenshots/06-private-route-table.png) |
| EC2 instances | [`07-ec2-instances.png`](screenshots/07-ec2-instances.png) |
| Web security group rules | [`08-web-security-group-rules.png`](screenshots/08-web-security-group-rules.png) |
| Database security group configuration | [`09-database-security-group.png`](screenshots/09-database-security-group.png) |
| Private security group rules | [`10-private-security-group-rules.png`](screenshots/10-private-security-group-rules.png) |
| Working website | [`11-working-website.png`](screenshots/11-working-website.png) |
| Session Manager session | [`12-session-manager.png`](screenshots/12-session-manager.png) |
| Private HTTPS connectivity test | [`13-private-https-test.png`](screenshots/13-private-https-test.png) |

## Skills Demonstrated

- Amazon VPC and IPv4 CIDR planning.
- Public and private subnet design across Availability Zones.
- Internet Gateway and NAT Gateway routing.
- Route tables and subnet associations.
- EC2 instance deployment using Amazon Linux 2023.
- Apache web server installation and basic website hosting.
- Stateful security groups and security group references.
- IAM instance roles and Systems Manager Session Manager.
- DNS resolution and outbound HTTPS connectivity testing.
- Network troubleshooting using observed test results.
- Git/GitHub documentation and infrastructure cleanup.
- AWS cost awareness and resource lifecycle management.

## Conclusion

This project connected foundational AWS networking concepts with practical cloud operations: IP addressing, subnet segmentation, routing, internet egress, EC2 web hosting, security groups, private-instance administration, troubleshooting, and cost-conscious teardown.

**Project status:** Infrastructure deployment and the documented connectivity tests were completed; the AWS resources were subsequently deleted after testing.

<p align="center">
  <strong>AWS Cloud Networking | Security | Connectivity | Troubleshooting | Cost Awareness</strong>
</p>

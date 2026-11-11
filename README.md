# Secure AWS Three-Tier Architecture

## 1. Project Overview

This project demonstrates the design, implementation, and
security validation of a three-tier application architecture
within Amazon Web Services (AWS).

The architecture separates the presentation, application,
and database layers using network segmentation,
least-privilege access controls, encrypted database
connections, and centralized secrets management.

The solution was deployed in a dedicated AWS lab
environment and validated through end-to-end
application connectivity and security testing. 

### Project Objectives

- Design a secure and scalable three-tier AWS architecture.
- Implement network segmentation across public and private subnets.
- Restrict communication between application components.
- Apply least-privilege IAM and database permissions.
- Protect database credentials using AWS Systems Manager Parameter Store.
- Encrypt application-to-database communications using TLS.
- Validate application functionality and security controls.

### Architecture Overview

The solution consists of three primary tiers:

**Presentation Tier**
- An internet-facing Application Load Balancer (ALB)
  receives incoming HTTP requests.
- The ALB forwards requests to the application tier.

**Application Tier**
- An Amazon EC2 instance operates within a private subnet.
- A Python application processes incoming requests.
- An IAM role provides controlled access to AWS services.
- AWS Systems Manager Parameter Store securely stores
  the database password.

**Database Tier**
- Amazon RDS for MySQL operates within private database subnets.
- Database access is restricted to the application tier.
- TLS protects application-to-database communications.
- A read-only database account enforces least privilege.

### End-to-End Application Flow

1. A user sends an HTTP request to the Application Load Balancer.
2. The ALB forwards the request to the private EC2 application server.
3. The application retrieves its database credential from Parameter Store.
4. The application establishes a TLS-encrypted connection to Amazon RDS.
5. Amazon RDS returns the requested records.
6. The application generates an HTML response.
7. The ALB delivers the webpage to the user.

### Implementation Outcome

The application was successfully deployed and validated.

End-to-end testing confirmed that requests routed through
the Application Load Balancer reached the private EC2
application server, which retrieved and displayed records
from the private Amazon RDS database.


---

## 2. AWS Architecture Components

The architecture was deployed in a dedicated AWS account
using the US East (N. Virginia) region.

| AWS Component | Configuration | Architectural Purpose |
|---|---|---|
| Amazon VPC | Dedicated VPC | Provides an isolated virtual network |
| Availability Zones | Two AZs | Distributes network resources across separate AZs |
| Public Subnets | Two | Host the internet-facing Application Load Balancer |
| Private Application Subnets | Two | Provide isolated network segments for application workloads |
| Private Database Subnets | Two | Provide isolated network segments for database resources |
| Internet Gateway | Attached to VPC | Enables internet routing for public subnets |
| Route Tables | Public, private application, private database | Control network traffic routing |
| Application Load Balancer | Internet-facing ALB | Receives and forwards application requests |
| Amazon EC2 | Amazon Linux 2023 | Hosts the Python application |
| Amazon RDS | MySQL 8.4 | Provides a managed relational database |
| AWS IAM | EC2 instance role | Provides temporary credentials for AWS API access |
| AWS Systems Manager | Session Manager | Enables administrative access without public SSH |
| Parameter Store | SecureString parameter | Stores the application database password |
| VPC Interface Endpoints | Systems Manager endpoints | Enable private connectivity to supported AWS services |
| Amazon EBS | EC2 root volume | Provides persistent storage for the application server |

### Network Architecture

The VPC contains six subnets distributed across two
Availability Zones:

- 2 public subnets
- 2 private application subnets
- 2 private database subnets

The Application Load Balancer is deployed across
the public subnets.

The EC2 application server is deployed in a private
application subnet without a public IP address.

The RDS database is deployed using a database subnet
group containing private database subnets.

This design separates internet-facing resources
from application and database workloads.

---

## 3. Security Architecture and Controls

Security controls were implemented across the network,
identity, application, database, and management layers.

| Security Domain | Implemented Control | Security Objective |
|---|---|---|
| Network Security | Public and private subnet segmentation | Reduce direct exposure of application and database resources |
| Network Security | Security group restrictions | Limit communication to authorized components |
| Identity and Access Management | EC2 IAM role | Avoid long-term AWS access keys |
| Least Privilege | Restricted Parameter Store IAM policy | Limit access to the required database credential |
| Secrets Management | Systems Manager Parameter Store SecureString | Protect database credentials |
| Encryption in Transit | TLS with RDS CA certificate verification | Protect database communications |
| Database Security | Read-only MySQL account | Restrict database modification privileges |
| Operating System Security | Dedicated non-root Linux service account | Reduce application process privileges |
| Application Security | Restricted systemd capabilities | Permit binding to port 80 without running the application as root |
| Infrastructure Security | Private EC2 without a public IP | Reduce direct internet exposure |
| Administrative Access | AWS Systems Manager Session Manager | Avoid exposing SSH access to the internet |

### Security Group Architecture

Three primary security groups control application traffic.

| Security Group | Inbound Rule | Source |
|---|---|---|
| ALB Security Group | HTTP 80 | Internet |
| Application Security Group | HTTP 80 | ALB Security Group |
| Database Security Group | MySQL 3306 | Application Security Group |

The ALB security group also permits inbound HTTPS
traffic on port 443; however, an HTTPS listener
was not configured during this lab.

The database security group does not permit
direct internet access.

### Identity and Access Management

The EC2 application server uses an IAM instance role
rather than long-term AWS access keys.

The role includes:

- AmazonSSMManagedInstanceCore for Systems Manager functionality.
- A custom policy permitting access to the required
  Parameter Store credential.

The application retrieves its database password
at runtime using AWS API calls.

### Database Security

Amazon RDS for MySQL was configured with:

- No public accessibility.
- A private database subnet group.
- Security group access restricted to the application tier.
- TLS-encrypted application connections with CA verification.
- A dedicated read-only application database account.

The application account was granted SELECT permissions
on the required table.

A database permission test confirmed that SELECT
operations succeeded while UPDATE operations were denied.

### Operating System Security

The Python application runs under a dedicated
non-root Linux service account.

A systemd service manages application startup
and restart behavior.

The service uses a restricted Linux capability
to bind to HTTP port 80 without running as root.

---

## 4. Security Architecture Considerations

This project demonstrates security controls within
a functional AWS lab environment.

The following improvements would be required
before treating the design as production-ready:

1. Configure HTTPS on the Application Load Balancer
   using AWS Certificate Manager.
2. Redirect HTTP traffic to HTTPS.
3. Replace Python's built-in HTTP server with
   a production-grade application server.
4. Deploy application instances across multiple
   Availability Zones for high availability.
5. Implement centralized logging, monitoring,
   and alerting.
6. Define backup, recovery, and disaster recovery
   requirements.
7. Implement infrastructure as code for
   repeatable deployments.

These improvements represent future architecture
enhancements rather than controls implemented
in the current lab.


---

## 5. Security Testing and Validation

Security validation was performed to verify the
effectiveness of network segmentation, identity
controls, secrets management, database permissions,
and application connectivity.

### Security Test Results

| Test ID | Security Control | Test Performed | Expected Result | Actual Result | Status |
|---|---|---|---|---|---|
| ST-01 | Network Segmentation | Verified EC2 was deployed in a private subnet without a public IP | EC2 is not directly internet-facing | Private EC2 configuration confirmed | PASS |
| ST-02 | Database Isolation | Verified RDS public accessibility settings | Database is not publicly accessible | Public accessibility disabled | PASS |
| ST-03 | Security Groups | Reviewed application security group inbound rules | HTTP traffic restricted to ALB security group | Inbound port 80 restricted to ALB security group | PASS |
| ST-04 | Database Security Groups | Reviewed RDS inbound rules | MySQL access restricted to application security group | Port 3306 restricted to application security group | PASS |
| ST-05 | IAM Role Access | Retrieved authorized Parameter Store parameter using EC2 IAM role | Authorized request succeeds | Parameter retrieval succeeded | PASS |
| ST-06 | Least-Privilege IAM | Reviewed custom Parameter Store policy | Permission scoped to required parameter | Resource-specific policy configured | PASS |
| ST-07 | Secrets Management | Retrieved database credential from Parameter Store at runtime | Application authenticates without hardcoded database password | Runtime credential retrieval succeeded | PASS |
| ST-08 | Encryption in Transit | Connected to RDS using TLS with CA certificate verification | Encrypted database connection succeeds | TLS connection established successfully | PASS |
| ST-09 | Database Least Privilege | Executed SELECT using read-only database account | SELECT operation succeeds | SELECT succeeded | PASS |
| ST-10 | Database Least Privilege | Attempted UPDATE using read-only database account | UPDATE operation denied | UPDATE denied | PASS |
| ST-11 | OS Least Privilege | Verified application runs under dedicated Linux account | Application runs without root privileges | Non-root service configured and running | PASS |
| ST-12 | Application Health | Requested application health endpoint | HTTP 200 response | Health endpoint returned HTTP 200 | PASS |
| ST-13 | End-to-End Connectivity | Accessed application through ALB DNS address | Application retrieves and displays RDS records | Both database records displayed successfully | PASS |

### Validation Summary

All 13 documented validation checks passed.

Testing confirmed that:

- Application and database resources were deployed
  within private network segments.
- Security groups restricted communication between tiers.
- The application accessed AWS services through
  an IAM instance role.
- Database credentials were retrieved from
  Systems Manager Parameter Store.
- Database communications used TLS with
  certificate verification.
- Database permissions restricted unauthorized
  modification operations.
- The application operated under a dedicated
  non-root Linux service account.
- End-to-end application connectivity was successful.

### Security Testing Limitations

The validation activities demonstrate that selected
security controls functioned as intended within
the lab environment.

They do not represent a comprehensive penetration
test, formal compliance assessment, or production
security certification.

Additional testing would be required for production
deployment, including vulnerability scanning,
security monitoring, resilience testing,
and broader IAM permission analysis.

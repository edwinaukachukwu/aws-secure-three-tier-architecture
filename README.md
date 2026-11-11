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

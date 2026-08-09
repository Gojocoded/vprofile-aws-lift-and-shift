 VProfile AWS Lift and Shift

📌 Application Overview

VProfile is a Java-based web application built using a multi-tier architecture.

This project focuses on migrating the existing application from a traditional server-based environment to AWS using a **Lift and Shift** approach.

The application consists of multiple components such as the web/application layer, database, caching, messaging, and search services.

The AWS deployment uses multiple EC2 instances and AWS managed services to provide better scalability, availability, security, and infrastructure management.

🛠️ Application Technologies

The VProfile application uses the following technologies:

- Java
- Spring MVC
- Spring Security
- Spring Data JPA
- JSP
- Maven
- Tomcat
- MySQL
- Memcached
- RabbitMQ
- Elasticsearch

🏗️ AWS Architecture

The application is deployed on AWS using a multi-tier architecture.
<p align="center">
  <img src="architecture/architecture-diagram.png" alt="VProfile AWS Lift and Shift Architecture" width="90%">
</p>
 Architecture Flow

```text
User
  ↓
Route 53
  ↓
Application Load Balancer (ALB)
  ↓
Auto Scaling Group (ASG)
  ↓
Tomcat Application Instances
  ↓
Backend Services
  ├── MySQL
  ├── Memcached
  ├── RabbitMQ
  └── Elasticsearch

S3 is used for storing build artifacts, while ACM provides the SSL/TLS certificate for HTTPS.

Security Groups are configured to control communication between the different application tiers.


**How the pieces fit together:**

- Users hit a custom domain (via **GoDaddy DNS**) which points to an **Elastic Load Balancer (ALB)** — replacing the original Nginx reverse proxy.
- The ALB terminates HTTPS using a certificate issued by **AWS Certificate Manager (ACM)** and forwards traffic to an **Auto Scaling Group** of **Tomcat** EC2 instances running the application.
- The Tomcat tier talks to backend services, each on its own EC2 instance: **MySQL** (database), **Memcached** (caching), **RabbitMQ** (messaging), and **Elasticsearch** (search).
- All internal service discovery runs through a **private Route 53 hosted zone**, so the app connects to friendly hostnames instead of hardcoded IPs.
- Build artifacts live in an **S3 bucket** and get pulled onto the Tomcat instance during deployment.
- Access between tiers is locked down with dedicated **Security Groups** per layer (`vprofile-ELB-SG`, `vprofile-app-sg`,vprofile-backend-ag`)


```
☁️ AWS Services Used

| AWS Service | Purpose |
|---|---|
| **Amazon EC2** | Hosts Tomcat, MySQL, Memcached, RabbitMQ, and Elasticsearch as individual instances |
| **Elastic Load Balancer (ALB)** | Replaces the original Nginx load balancer, distributes traffic, and terminates SSL |
| **Auto Scaling Group (ASG)** | Automatically scales the Tomcat application instances based on demand |
| **Amazon Route 53** | Provides private DNS for internal service communication and public domain mapping |
| **Amazon S3** | Stores build artifacts used during deployment |
| **AWS Certificate Manager (ACM)** | Provides the SSL/TLS certificate for HTTPS on the load balancer |
| **AWS IAM** | Provides scoped permissions for accessing AWS resources such as S3 |
| **Security Groups** | Controls network access between the ELB, application, and backend tiers |



 ⚙️ Deployment Flow

1. Log in to AWS account
2. Create a key pair for SSH access (`vprofile-prod-key`)
3. Create security groups for each tier (ELB, app, backend)
4. Launch EC2 instances using bootstrap (user data) bash scripts
5. Map private IPs to friendly names in a private Route 53 hosted zone
6. Build the application from source (Maven → WAR file)
7. Upload the build artifact to an S3 bucket
8. Pull the artifact down onto the Tomcat EC2 instance and deploy
9. Set up the ELB with HTTPS using an ACM certificate
10. Point the public domain (GoDaddy DNS) at the ELB endpoint
11. Verify the application is reachable end-to-end
12. Wrap the Tomcat instance in an Auto Scaling Group for elasticity
    

🗄️ Database

The application uses **MySQL** as the relational database.

The database schema and initial data are provided in the SQL dump file:

`src/main/resources/db_backup.sql`

To import the database dump into MySQL:

```bash
mysql -u <username> -p accounts < db_backup.sql

📁 Repository Structure

```text
vprofile-aws-lift-and-shift/
├── ansible/
├── src/
├── userdata/
├── Jenkinsfile
├── README.md
├── pom.xml
└── ai2023rmq.repo

🚀 Project Highlights

- Migrated the VProfile application from a traditional server-based setup to AWS.
- Implemented a multi-tier AWS architecture for better scalability and separation of responsibilities.
- Replaced the original Nginx load balancer with an AWS Application Load Balancer.
- Used an Auto Scaling Group for the Tomcat application tier.
- Configured private Route 53 DNS for internal service communication.
- Used S3 for storing build artifacts during deployment.
- Enabled HTTPS using AWS Certificate Manager.
- Applied Security Groups to control communication between application tiers.

 ⚠️ Challenges & Troubleshooting

During the deployment, several issues were encountered and resolved:

### 1. Connection Timeout

The application was initially not reachable because the required inbound traffic was not allowed through the appropriate Security Group.

**Resolution:** Reviewed and corrected the Security Group rules across the ELB, application, and backend tiers.

### 2. HTTP 500 — Java Version Compatibility

Tomcat returned an `UnsupportedClassVersionError` because the application was built with a newer Java version than the Java runtime available on the Tomcat instance.

**Resolution:** Checked the Java versions and aligned the runtime with the version used to build the application.

### 3. Tomcat Service Not Found

The expected `tomcat` service name was not available on the instance.

**Resolution:** Identified the correct service name for the installed Tomcat package and used the appropriate systemctl command.

### 4. Load Balancer Connection Refused

The application initially returned a connection-refused error through the load balancer.

**Resolution:** Checked the target group and health-check configuration and traced the issue through the load-balancing layer.

### 5. Credentials in application.properties

Sensitive database and messaging credentials had to be removed before publishing the project publicly.

**Resolution:** Replaced real credentials with placeholders before committing the configuration to the public repository.

 🎯 Result

The VProfile application was successfully deployed on AWS using a Lift and Shift approach.

The final deployment runs the application behind an AWS Application Load Balancer with HTTPS enabled. The Tomcat application tier communicates with the backend services — MySQL, Memcached, RabbitMQ, and Elasticsearch — through the AWS network and private DNS configuration.

The deployment demonstrates how a multi-tier application can be migrated to AWS with minimal changes to the existing application architecture.

📚 What I Learned

- **AWS Networking** — Security Groups, Application Load Balancer, target groups, and health checks
- **Linux Administration** — Managing services, packages, Java runtime, and application servers on EC2
- **Tomcat Deployment** — Deploying and troubleshooting a Java WAR application on EC2
- **Maven** — Building the application and generating a deployable WAR file
- **AWS Storage** — Using S3 to store and retrieve application build artifacts
- **DNS** — Using Route 53 for private service discovery and domain mapping
- **HTTPS** — Configuring SSL/TLS using AWS Certificate Manager
- **Load Balancing & Auto Scaling** — Using AWS services to improve application availability and scalability
- **Git & GitHub** — Managing the project repository and preparing configuration safely for a public repository
- **Troubleshooting** — Identifying and resolving issues across the network, infrastructure, runtime, and application layers

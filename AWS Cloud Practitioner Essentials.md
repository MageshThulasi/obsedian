Amazon Elastic Compute Cloud - Amazon EC2
- Multi-tenancy, Hypervisor
- Compute-as-a-Service model
Elastic Load Balancer - ELB
Amazon Simple Queue Service - SQS
- Send/Store messages
Amazon Simple Notification Service - SNS
- Public/Subscribe model with Topic

Serverless computing
- AWS Lambda - Lambda function (configure Trigger) - Run under 15 mins
- AWS Container Services
	- Amazon Elastic Container Services - Amazon ECS
	- Amazon Elastic Kubernetes Services - Amazon EKS
	- You can run ECS or EKS on
		- EC2 instances that you manage or
		- AWS Fargate Serverless instances

AWS Global Infrastructure
- Region, Availability Zone
- Edge locations run
	- Amazon CloudFront, which are CDNs
	- Amazon Route 53 - DNS service
- AWS Outposts

Provision AWS services
- Using APIs
- AWS Management Console
- AWS CLI (Command Line Interface) - using Terminal of your machine
- AWS SDKs (Software Development Kit)
- AWS Elastic Beanstalk
- AWS CloudFormation
	- Infrastructure as Code (IAC)

Virtual Private Cloud (VPC) & Subnets
- Public Subnet
- Private Subnet
- Public Traffic -> Internet Gateway (IGW) -> Resources on the Public Subnet
- Private Traffic -> Virtual Private Gateway -> Resources on the Private Subnet
- AWS Direct Connect
- Network ACL (Access Control List) is Stateless - is at the Subnet level
- Security Group is Stateful - is at the Resource level
- Amazon Route 53 - DNS service
	- Translates website name to IP address
	- Routing policies - Latency-based routing, Geolocation DNS, Geoproximity routing, Weighted round robin
- Amazon CloudFront - CDN
	- Edge location

Databases and storage
- Amazon EBS (Elastic Block Store) volumes
	- Volumes attach to EC2 instances
	- Availability Zone level resource
	- Need to be in the same AZ to attach EC2 instances
	- Volume do not automatically scale
- Amazon S3 (Simple Storage Service)
	- Store data as objects (max object size is 5 TB)
	- Store objects in buckets
	- Amazon S3 Standard
	- Amazon S3 Standard-IA (Infrequent Access) for backups or disaster recovery files
	- Amazon S3 Glacier Flexible Retrieval
	- You can create lifecycle policies to move data between these storage options
- Amazon EFS (Elastic File System)
	- Linux file system
	- Regional resource
	- Automatically scales
- Amazon RDS (Relational Database Service)
- Amazon Aurora
	- MySQL, PostgreSQL
- Amazon DynamoDB
	- A serverless database
	- Is a NoSQL or non-relational database
	- Fully managed and highly scalable
	- Millisecond response time
- Amazon Redshift
	- Data wareshousing as a service
	- Big data BI (Business Intelligence) solutions
- AWS DMS (Database Migration Service)
	- Homogeneous migration (Example - MySQL to Amazon RDS MySQL)
	- Heterogeneous migration
		- Step 1: AWS Schema Conversion Tool
		- Step 2: AWS DMS to migrate from source to target database

User Permissions and Access
- Root account user
- AWS IAM (Identity Access and Management)
	- Principle of least privilege
	- IAM Policy
		- Statement: Allow or Deny
		- Action: Any AWS API call 
		- Resource: Which AWS resource the API call is for
	- IAM Groups
		- Groupings of user
	- Attach a policy to group and assign users to groups
	- Roles
		- Access to temporary permissions
		- Assign roles to AWS resources, users, applications, etc.

AWS Organizations
- A central location to manage multiple accounts
- Consolidated billing
- Hierarchical groupings of accounts
- AWS Service and API actions access control

AWS Artifacts - Compliance
Distributed Denial of Service (DDoS) attacks
AWS Shield with AWS WAF
AWS KMS (Key Management Service)
Amazon Inspector - runs security assessment
- Network configuration reachability piece
- Amazon agent
- Security assessment service
Amazon GuardDuty

Monitoring and Analytics
- Amazon CloudWatch
	- Custom Metrics
	- Amazon CloudWatch Alarm
	- Dashboard
	- Reduce MTTR and improve TCO
- AWS CloudTrail
	- API auditing tool
- AWS Trusted Advisor
	- Cost Optimization
	- Performance
	- Security
	- Fault Tolerence
	- Service limits

Pricing and Support
- AWS Free Tier
	- Always Free - Lambda, S3, Lightsail, DynamoDB, SNS, Cognito
	- 12 months free
	- Trials
- Billing Dashboard
- Consolidated Billing
- AWS Budgets
- AWS Cost Explorer
- AWS Support Plans
	- Basic, Developer, Business
	- Enterprise On-Ramp, Enterprise Support - TAM (Technical Account Manager)
	- Well-Architected Framework
- AWS Marketplace

Migration and Innovation
- AWS CAF (Cloud Adoption Framework)
	- Business Perspective
	- People Perspective
	- Governance Perspective
	- Platform Perspective
	- Security Perspective
	- Operations Perspective
- Migration Strategies
	- Rehosting - Lift-and-shift
	- Replatforming - Lift-tinker-and-shift
	- Refactoring/re-architecting - highest initial cost
	- Repurchasing
	- Retaining
	- Retiring
- AWS Snow Family
	- AWS Snowcone - 8 TBs
	- AWS Snowball - 80 TBs
	- AWS Snowmobile - 100 PBs
- Innovation with AWS
	- VMWare cloud on AWS
	- Amazon SageMaker - Machine Learning models
	- Amazon Augmented AI - A2I
	- Amazon Lex - chat bots
	- Amazon Textract
	- AWS DeepRacer
	- AWS Ground Station - Satellite

The Cloud Journey
- The Well-Architected Framework
	- Operational Excellence
	- Security
	- Reliability
	- Performance Efficiency
	- Cost Optimization
	- Sustainability
- AWS Well-Architected Tool
- 6 main benefits of AWS cloud
	- 
	- Massive economies of scale
	- Stop guessing capacity
	- Increase speed and agility
	- Stop spending money running and maintaining data centers
	- Go global in minutes

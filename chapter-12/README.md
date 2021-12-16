# The Security Pillar

- To effectively protect data, you need to ensure three elements of the data:
  - **Confidentiality** - only people or systems that can access data are those authorised to access it, which can be implemented using encryption and access control lists (ACLs)
  - **Integrity** - the data has not been maliciously or accidentally changed, which can be verified using Cryptographic hashing and logging
  - **Availability** - the data is available to those who need it when they need it



## Identity and Access Management

- AWS credentials are the keys to the kingdom, and hence the first order of business is to protect them from accidental exposure and unauthorised use
- The second step is to ensure that users have only the permissions they need, and no more

### Protected AWS Credentials

- The root user has unrestricted access to your AWS account, so it’s a best practice to avoid using it for routine administrative tasks
- A crucial step in securing the root user is to enable multi-factor authentication (MFA)
- For IAM users, you can choose to enforce a password policy to ensure that all IAM users with access to the AWS Management Console have a secure password

### Fine-Grained Authorisation

- You should give IAM principals permissions to only the resources they need and no more - by default IAM principals have no permissions, so creating an account will mean they can do nothing.
- IAM users and roles derive their permissions only from IAM policies. A policy contains one or more permission statements, which consist of:
  - **Effect** - Allow or Deny
  - **Action/Operation** - a set of actions or operations that it can perform on its resources, e.g. `EC2:RunInstances`
  - **Resource** - the resource(s) that the actions can be performed against
  - **Condition** - additional conditions that must be met in order for this permission to be granted, e.g source IP Address

- AWS provides hundreds of prepackaged policies called *AWS managed policies*
- A *customer-managed policy* is a stand-alone policy that you create and can attach to principals in your AWS account
- An *inline policy* is a set of permissions that’s embedded in an IAM principal or group

### Permissions Boundaries

- Permission boundaries let you limit the maximum permissions an IAM principal can be assigned, which prevents you from accidentally giving a user too many permissions by inadvertently attaching the wrong permissions policy
- The permissions boundary limits the user to performing only those actions laid out in the permissions boundary policy, regardless of any additional policies that are attached (i.e. it overrides any attached policies)
- For example attaching an AdministratorAccess policy to a principal that has a permission boundary that only allows EC2 access will result in the principal only being able to perform EC2 actions, not anything else contained within the AdministratorAccess policy.

### Roles

- A *role* is an IAM principal that doesn’t have a password or access key
- An IAM user or AWS resource can assume a role and inherit the permissions associated with that role
- An IAM user can assume a role which allows the user to only have the permissions that are defined within the role

### Enforcing Service-Level Protection

- In addition to defining identity-based IAM policies to control access to resources, some AWS services allow you to define resource-based policies - such as S3 bucket policies, KMS policies to specify the admins of a given key
- Users *without* AWS credentials tend to consume the AWS services that offer resource-based policies — examples include receiving an SNS notification via email, or downloading a publicly accessible file from S3 even though the user doesn't have an AWS account
- As with identity-based policies, use the principle of least privilege when creating resource-based policies



## Detective Controls

- AWS offers a number of detective controls that can keep a record of the events that occur in your AWS environment, as well as alert you to security incidents or potential threats.

### CloudTrail

- CloudTrail can log activities on your AWS account, which include:
  - Management events and/or data events
  - Read-only and/or write-only events
  - All resources or just specific ones
  - All regions or just specific regions
  - Global services
- All logs are stored in S3, so access to these logs will need to be locked down.
- Log files should be encrypted (with SSE-KMS) and have log file integrity turned on
- It can take up to 15 minutes for logs to be written to the S3 bucket

### CloudWatch Logs

- CloudWatch Logs can aggregate logs from multiple sources for easy storage and searching, such as:
  - **CloudTrail Logs**
  - **VPC Flow Logs** — for monitoring traffic entering or leaving a VPC (not including DHCP or DNS traffic)
  - **RDS Logs**
  - **Route 53 DNS Queries**
  - **Lambda**
- CloudWatch logs can be queried using a proprietary JSON structure, although this is limited
- Logs stored in S3 can be searched with Amazon Athena using SQL queries. 
  - When importing the data into Amazon Athena you will need to define the structure of the data using a DDL statement (which AWS can provide automatically if required)
  - Athena supports CSV, TSV, JSON and Apache ORC and Parquet file types

### Auditing Resource Configurations with AWS Config

- AWS Config can alert you when a resource configuration in your AWS account changes
- It can also compare your resource configurations against a baseline and alert you when the configuration deviates from it - which is useful for ensuring your environment maintains certain levels of compliances
- AWS Config Rules let you define configuration states that are abnormal or suspicious — for example alerting you if an EBS volume is not attached to an instance
- For each resource, AWS Config can show a configuration timeline which shows each time a resource was modified — for example when an EBS volume was attached and detached from an instance

### Amazon GuardDuty

- Amazon GuardDuty looks for security threats by inspecting network traffic to and from your instances
- GuardDuty analyses VPC flow logs, CloudTrail management event logs, and Route 53 DNS query logs looking for known malicious IP address, domain names, and any potentially malicious activity
- Each GuardDuty finding is displayed in the Console (as well as delivered to CloudWatch Events), can be one of the following types:
  - **Backdoor** — indicates that an EC2 instance has been compromised by malware that can be used to spend spam or participate in a DDoS.
  - **Behaviour** — an EC2 instance is communicating of a protocol/port that it normally doesn't or is sending a large amount of traffic to an external host
  - **Cryptocurrency** — an EC2 instance is producing activity that is linked to a Bitcoin node
  - **Pentest** — an EC2 instance running linux distributions used for penetration testing is making API calls against your AWS resources
  - **Persistence** — an IAM user (with no prior history) has modified user or resource permissions, security groups, or network ACLs
  - **Policy** — root user credentials were used or S3 block public access was disabled
  - **Recon** — indicates a reconnaissance attack (an attack to gain information that is used in subsequent access or DoS attacks) may be underway
  - **ResourceConsumption** — an IAM user has launched an EC2 instance, despite having no history of doing so
  - **Stealth** — a password policy was weakened, CloudTrail logging was disabled or modified, or CloudTrail logs were deleted
  - **Trojan** — an EC2 instance is exhibiting behaviour that indicates a Trojan (a program that leaks data to an attacker) may be installed
  - **UnauthorisedAccess** — indicates a possible unauthorised attempt to access your AWS resources via an API call, a console login or by brute forcing an SSH or RDP session
- Finding types will relate to either the inappropriate use of AWS credentials or the presence of malware on an EC2 instance

### Amazon Inspector

- Amazon Inspector is an agent-based service that looks for vulnerabilities on your EC2 instances
- The Inspector agent runs an *assessment* on the instance and analyses its network, filesystem, and process activity, determining whether any threats or vulnerabilities exist based upon the configured rules.
- Inspector offers five rule packages:
  - **Common Vulnerabilities and Exposures (CVEs)** — common vulnerabilities found in publicly released software
  - **Centre for Internet Security Benchmarks** — security best practises for Linux and Windows
  - **Security Best Practices** — provides a handful of best practice rules against Linux instances only, looking for issues such as root access via SSH or insecure permissions on system directories
  - **Runtime Behaviour Analysis** — detects the use of insecure client and server protocols, unused listening TCP ports and inappropriate file permissions
  - **Network Reachability** — rules that detect network configurations that make resources in your VPC vulnerable

- After an assessment runs, Inspector generates a list of findings classified by the following severity levels:
  - **High** — the issue should be resolved immediately
  - **Medium** — the issue should be resolved at the next possible opportunity
  - **Low** — the issue should be resolved at your convenience
  - **Informational** — the issue is unlikely to create a security risk, but it should be reviewed

- After you resolve any outstanding security vulnerabilities on an instance, you can create a new AMI from that instance and use it going forward when provisioning new instances.

### Amazon Detective

- Amazon Detective is designed to help you correlate events and see how a given event affects particular resources - which is easier than having to manually digging through CloudTrail logs
- Amazon Detective takes information from VPC flow logs, CloudTrail, and GuardDuty and places this information into a graph database
- You can visualise the graph model to correlate events to identify and investigate suspicious or interesting activities against your AWS resources

### Security Hub

- Security Hub is a one-stop shop for the security status of your entire AWS environment
- It collects security information from Amazon Inspector, GuardDuty, and Macie (see Data Encryption), as well as assessing your account against AWS security best practices and PCI DSS standards
- All findings are displayed in a user-friendly dashboard that contains charts and tables



## Protecting Network Boundaries

- Your network should perform part of your defensive strategy, and thus designing networks in a secure manner is important
- AWS services that are publicly available are the responsibility of AWS, however access to your network is your responsibly

### NACLs and Security Groups

- Network ACLs define what traffic is allowed to and from a subnet
- Security groups provide granular control of traffic to and from individual resources, such as instances and elastic load balancer listeners
- You should configure your security groups and network ACLs to allow traffic to and from your AWS resources using only the protocols and ports required
- Private subnets should not have a default route, nor should have an internet gateway

### AWS Web Application Firewall

- The Web Application Firewall (WAF) monitors HTTP and HTTPS requests to an application load balancer or CloudFront distribution
- WAF will protect your applications from common exploits which could result in a DoS attacks
- WAF allows you to inspect your application traffic for malicious scripts, very long query strings, SQL injection or cross-site scripting and block these requests so that they never reach your application
- WAF can block based on source IP patterns or geographic location, as well as supporting Lambda functions to automatically update the IP address list if required (by analysing web traffic using a Lambda function)

### AWS Shield

- AWS Shield is a service that helps protect your applications from DoS attacks, and comes in two flavours:
  - **AWS Shield Standard** — defends against Layer 3/4 DDoS attacks (such as SYN flood and UDP attacks) and is automatically activated for all AWS customers
  - **AWS Shield Advanced** — builds on top of the Standard option, and adds Layer 7 attack support (HTTP flood attacks by sending large HTTP GET requests), as well as forensic reports, and 24/7 from the AWS DDoS response team
- AWS Shield mitigates:
  - 99% of all attacks within five minutes or less
  - CloudFront and Route 53 attacks in less than one second
  - Elastic Load Balancing in less than five minutes
  - All other service attacks within twenty minutes



## Data Encryption

### Data at Rest

- Majority of data will be stored in S3, EBS, EFS or RDS all of which integrate with KMS, allowing you to encrypt data with your own customer-managed customer master key (CMK) or an AWS-managed CMK.
- Keys can be rotated, disabled or revoked, and AWS-managed keys are rotated once a year automatically (this cannot be disabled)
- **S3** supports a number of encryption option include SSE-S3, SSE-KMS, SSE-C and Client-side encryption, and encryption is applied per objects rather than at a bucket level.
- **EBS** can be encrypted using a KMS-managed key
- **EFS** can be encrypted using a KMS customer managed key to encrypt the file system metadata and the data itself

### Data in Transit

- Encrypting data in transit is enabled through a TLS certificate, which ca be obtained from the Amazon Certificate Manager (or use your own)
- Private keys generated by ACM cannot be exported, and thus cannot be installed into an EC2 instance or on-premise server

### Macie

- Macie is a service that automatically locates and classifies your sensitive data stored in S3 buckets and shows you how it’s being used
- Macie will also perform the following tasks:
  - Recognising sensitive data like trade secrets and personally identifiable information 
  - Alerting you if it finds buckets that have permissions that are too lax
  - Tracking changes to bucket policies and ACLs
  - Classifying other types of data based on custom data identifiers that you define

- Macie classifies its findings into either policy findings (such as changing a bucket policy or removing encryption) or sensitive data findings (such as finding sensitive data in an S3 bucket)
- All sensitive data findings will be automatically published, while policy finds are published every 15 minutes
- All findings are published to EventBridge and the AWS Security Hub

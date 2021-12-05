# The Reliability Pillar

- Reliability — also known as *resiliency* — is the ability of an application to avoid failure and recover from it quickly when it occurs
- Achieving a high degree of reliability for your application is not a trivial matter and entails significant cost and complexity



## Calculating Availablity

- Availability is the percentage of time an application is working as expected — generally you will want application availbility of at least 99% and often higher
  - **99%** allows for up to 3 days, 15 hours and 39 minutes of downtime per year
  - **99.9%** allows for up to 8 hours and 45 minutes of downtime per year
  - **99.95** allows for up to 4 hours and 22 minutes 
  - **99.99** allows for up to 52 minutes of downtime per year
  - **99.999** allows for 5 minutes of downtime per year

- **Traditional Applications** — ones that are migrated from on-premise deployments straight to the cloud — generally require more resources in order for them to be highly available, notably because EC2 availability is lower than managed AWS services such as Lambda
- Spanning your application across multiple availability zones (and ideally regions) greatly increases your availability, but will also increase your costs, as you are running more instances
- Cloud resources are not unlimited, and thus there are limits to the number of resources per account. The AWS Trusted Advisor can be used to view which service limits are applied to your account, as well CloudWatch Alarms which can be used to notify you when you are close to hitting a limit
- Your application's design can also affect your availability score, for example a monolith application is much more prone to causing an outage in comparison to  a set of micro-services



## EC2 Auto Scaling

-  *EC2 Auto Scaling* service offers a way to both avoid application failure and recover from it when it happens, by provisioning and starting on your behalf a specified number of EC2 instances
- EC2 Auto Scaling uses either a *launch configuration* or a *launch template* to automatically configure the instances that it launches

### Auto Scaling Groups

- An *Auto Scaling group* is a group of EC2 instances that Auto Scaling manages using either the launch configuration or launch template to create each instance
- An Auto Scaling group requires a minimum (minimum number of instances to maintain), maximum (the maximum number of instances to maintain) and optionally the desired capacity (the optimum level of instances).
  - If desired capacity is given, then no matter what the minimum value is, AWS will always aim to keep the number of instances at the desired capacity level
- When using an ALB, the ALB target group must reference the Auto Scaling group, so that it when Auto Scaling creates a new instance, it will automatically add it to the ALB target group
- Auto Scaling groups will determine each instance's health based upon EC2 health checks, such as memory exhaustion, filesystem correction or networking configuration errors
- ALB can use HTTP health checks for the load balancer's target groups — responses must be within the 200 - 499 range — Auto Scaling groups can use these results to determine if an instance is healthy
  - Instances that are not healthy will have traffic routed away from them automatically by ALB, at the same time the instance will be replaced by Auto Scaling

#### Auto Scaling Options

- Auto Scaling provides several other options to scale out the number of instances to meet demand

##### Manual Scaling

- changing the minimum, desired or maximum values at any time, will cause Auto Scaling to reevaluate and adjust (if needed) immediately

##### Dynamic Scaling Policies

- To ensure that your instances never become overburdened, dynamic scaling policies automatically provision more instances *before* they run out of resources
- Dynamic Auto Scaling policies can track the following metrics
  - Aggregate CPU utilisation
  - Average request count per target
  - Average network bytes in
  - Average network bytes out

- You can also use custom metrics, such as creating instances if a particular operation is taking a longer than expected
- Dynamic scaling policies work by monitoring a CloudWatch alarm and scaling out — there are three types of dynamic scaling policies: *ChangeInCapacity*, *ExactCapacity* and *PercentChangeInCapacity*. 

##### Step Scaling Policies

- Using a *step scaling policy*, you can instead add instances based on how much the aggregate metric exceeds the threshold
- For example you specify a lower and upper bound (e.g. in-between 50% and 60% CPU usage), as well as a adjustment type (from the dynamic scaling policies) and the amount to increase by (e.g. 2 instances, or 20% of group size) and Auto Scaling will handle the actions by creating or removing instances ignorer to achieve the target in between the lower/upper bounds
- A warm up time can be configured which will be used by Auto Scaling to wait before applying the policies again.

##### Target Tracking Policies

- Target tracking policies allow Auto Scaling to track a target metric/value, and Auto Scaling will adjust the number of instances to keep the metric near the target
- The metric you choose must change proportionally to the instance load — examples include average CPU utilisation for the group, the number of SQS messages in a queue or the number of requests per instance

##### Scheduled Actions

- Scheduled actions are useful if you have a predictable load pattern and want to adjust your capacity proactively, ensuring you have enough instances *before* demand hits
- A scheduled action requires a minimum, maximum or desired capacity value, as well as a start date and time.
- Scheduled actions can be one off or set to occur at regular intervals, as well as being able to set an end time, in which the scheduled policy is removed (and all actions are reverted).



## Data Backup and Recovery

- Amazon takes great pains to ensure the durability of your data stored on it's storage services is not lost, however data loss is always a possibility

### S3

- All S3 storage classes except One Zone-Infrequent Access distribute objects across multiple availability zones - to avoid data loss in a single AZ you should look to use other storage classes
- S3 versioning should be enabled to guard against deletion and data corruption, as S3 never overwrites or deletes an object, instead it create a new version, allowing you to revert back if required
- Cross Region Replication should be used to protect your data against multiple availability zone failures, or the failure of an entire region. Note: deletions on the source bucket do not get deleted.

### Elastic File System (EFS)

- AWS Elastic File System (EFS) provides a managed Network File System (NFS) that can be shared across multiple servers
- EFS filesystems are stored across multiple zones in a region, and can withstand an availability zone failure
- To protect against data corruption, you can backup files to an S3 bucket, or another EFS filesystem in the same region (or use AWS Back Service to do this incrementally for you)

### Elastic Block Storage (EBS)

- EBS automatically replicates volumes across multiple availability zones in a region so that they’re resilient to a single availability zone failure.
- To avoid data corruption, you should take a backup of an EBS volume in the form of a Snapshot, which is stored on S3
- A Snapshot can be created manually, or automatically using the Amazon Data Lifecycle Manager

### Database Resiliency

- Running your own database will require you to export your database to a file and storing that on S3, or alternatively you could create a EBS Snapshot (assuming the latest data is flushed to disk)
- Amazon Relational Database Service (RDS) supports taking a database instance snapshot, which will remove any complexities in the backup procedure. Restoring a database snapshot will always create a new database instance
- Multi-AZ RDS deployments can replicate data from the Primary instance to all the Standby instances, so that if the Primary instance were to fail, then the standby databases have the latest data
- For maximum resiliency, use Amazon Aurora with multiple Aurora replicas. Aurora will store your data across three different availability zones - a failure in the primary instance, will allow one of the other Aurora replicas to become the primary instance
- DynamoDB stores tables across multiple availability zones, which provides speed and protected against an availability zone failure. DynamoDB global table can also be used to replicate your tables to different regions. Point-in-time recovery allows you to restore to any point within the last 35 days



## Creating a Resilient Network

- When it comes to network design, you must consider two elements: 
	1. your virtual private cloud (VPC) design and;
	2. how users will connect to your resources within your VPC

### VPC Design Considerations

- When creating a VPC, make sure you choose a sufficiently large CIDR block — the more redundancy you need the more IPs you'll need
- When creating your subnets, leave enough unused address space within the CIDR to add additional subnets later
- Consider creating separate subnets to segment your resources for security or ease of management, this is known as a *multi-tier architecture*

- EC2 instances aren’t the only resources that consume IP address space - ELBs, databases and VPC endpoints all require IP addresses

### External Connectivity

- If users can’t connect to your application because of a network issue, your application is unavailable to them, even if it’s healthy and operating normally, thus the method in which users connection to the AWS Cloud needs to meet your availability requirements
- The Internet is generally reliable, but may come at a cost of unpredictable speed and latency
- Consider using Direct Connect (Either as a primary connection or as a backup) for mission-critical applications
- Ensure that your VPC IP addresses do not overlap with IP addresses on an external network, such as when using Direct Connection, VPC peering or VPN connections


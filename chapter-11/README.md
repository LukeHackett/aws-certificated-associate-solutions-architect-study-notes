# The Performance Efficiency Pillar

- Squeezing the most performance out of your cloud resources is one of the pillars of the AWS Well Architected Model




## Optimising Performance for the Core AWS Services
### Compute

- When using EC2, you should look to match the instance type to as closely as possible the workload you would expect to face
- Each Instance type has a number of parameters that can be taken into consideration, such as:
  - ECU - EC2 Compute Units which can be used to compare two instances based upon their compute power
  - vCPUS, Clock Speed, Memory, Processor Architecture
  - Instance Storage, or EBS-Optimised Available
  - Network Performance, IPv6 Support
- Auto Scaling should be considered to allow for demand to be handled automatically by scaling in (or out) the number of instances
- Serverless workloads should be considered when compute should be used to response to a changing environment
  - Fargate, ECS and EKS can maximise your resource utilisation, by allowing you tightly pack multiple fast-loading workloads on a single instance
  - AWS Lambda works without the need to provision an ongoing server host, and are designed for running brief spurts of compute based upon some network event
- Common use cases for compute categories:
  - **EC2 instances**
    - Long-lived, more complex web application
    - Processes requiring in-depth monitoring and tracking
  - **ECS containers**
    - Highly scalable, automated applications
    - Applications requiring full control over underlying resources
    - Micro-services deployments
    - Testing environments
  - **Lambda functions**
    - Retrieving data from backend databases
    - Parsing data streams
    - Processing transactions

### Storage

- Software RAID can be used to optimise the performance and reliability of your disk-based data:
  - **RAID 0** segments data across multiple disks, allowing for concurrent operations, however it does not add durability to your data. This is recommended for heavy-transaction operations such as busy databases
  - **RAID 1** creates mirrored sets of data on multiple volumes, which doesn't improve the performance, but does add durability
  - **RAID 5 & RAID 6** configurations can combine both performance and reliability features, but because of IOPS consumption considerations, they’re not recommended by AWS
  - Note: AWS also does not recommend using RAID-configured volumes as instance boot drives since the failure of any single drive can leave the instance unbootable
- S3 Cross-Region Replication (CRR) can be used to automatically (asynchronously) sync contents of a bucket in one region with a bucket in a second region
  - You can choose to have all objects in your source replicated or only those whose names match a specified prefix
  - When an object is deleted from the source bucket, it will similarly disappear from the destination (unless versioning is enabled on the destination bucket)
- Amazon S3 Transfer Acceleration can be used to speed up transferring large files from your computer to S3, by routing data through the CloudFront edge locations. If you plan on using this, it can be enabled on the bucket that requires it
- S3 should also be considered to host static websites, or serve static content such as files, images or media.

### Database

- When designing your data management environment, you’ll have some critical decisions to make: should you build your own platform on an EC2 use for Amazon’s Relational Database Service (RDS)?
- A custom database setup allows you to have more control over a much wider range of options, such as:
  - **Consistency, Availability and Partition Tolerance (CAP)** - it is impossible to suffice all three of these, and thus sacrifices must be made in the design
  - **Latency** - the storage volume type (SSD or magnetic, number of IOPS etc) will greatly determine the read and write performance of your database
  - **Durability** - how will you protect your data from hardware failure (see chapter 10)?
  - **Scalability** - does the design allow for automated resource growth, such as using Auto Scaling
- Carefully configuring details such as schemas, indexes, views, option groups, and parameter groups can improve the performance of your databases (either custom or RDS hosted)
- Sometimes not using a database can also be considered, tools such as Amazon Redshift Spectrum, Athena, and Elastic Map Reduce (EMR) can effectively connect and consume data

### Network Optimisation and Load Balancing

- The geolocation and latency-based routing provided by Route 53 and CloudFront are important elements of a strong networking policy, as are VPC endpoints and the high-speed connectivity offered by AWS Direct Connect
- High-bandwidth EC2 instance types should be considered whereby node communication needs to be fast
  - By using the enhanced networking functionality, you can achieve you network speeds of up to 100 Gbps
  - There are three enhanced networking technologies: Intel 82599 Virtual Function (VF) interface, Elastic Network Adapter (ENA), and Elastic Fabric Adapter (EFA), each of which is only supported on certain instance types
- Load Balancing is one of the most important networking concepts, as it allows your application to serve lots of traffic, from multiple instances.
  - Application Load Balancer is used for traffic at layer 7 (i.e. HTTP and HTTPS) and supports path based routing, which is ideal for micro-services and other tiered architectures
  - Network Load Balancer is used for traffic for at layer 7 (i.e. TCP, UDP) and supports huge traffic volumes and spikes, as well as supporting VPN connections



## Infrastructure Automation

### CloudFormation

- You can represent infrastructure resource stacks using AWS CloudFormation as a JSON or YAML-formatted template file
- CloudFormation templates are easy to copy or export to reliably re-create resource stacks elsewhere
- A CloudFormation stack is the group of resources defined by your template - and AWS will report when the stack has successfully loaded, or roll back changes if a failure is detected

### Third-Party Automation Solutions

- You can manage your AWS resources using the AWS CLI (and creating scripts)

- Besides Bash and PowerShell scripts, third-party configuration management tools such as Puppet, Chef, and Ansible can be used to closely manage AWS infrastructure.
- **AWS OpsWorks: Chef** allows you to create a Stack and one or more layers. 
  - A layer is a definition that adds functionality to your stack, such as a Node.js application server, a load balancer, or a chef recipe
  - OpsWorks Chef deployments also include one or more apps, which points to the application code you want to install on the instances running in your stack
- **AWS OpsWorks: Puppet** allows you to launch an EC2 instance as a puppet master server, that you can use to deploy your application to puppet nodes.



## Reviewing and Optimising Infrastructure Configurations

- In order to achieve and maintain high-performance standards for your applications, you’ll need solid insights into how they’re doing.

### Loading Testing

- Load (or stress) testing will subject your infrastructure to simulated workloads and carefully observe how it handles the stress
- There are various load testing tools that are available via the AWS Marketplace
- Before launching some kinds of vulnerability or penetration tests against your infrastructure, for instance, you need to request explicit permission from AWS

### Visualisation

- Look to setup Simple Notification Service (SNS) messages or email alerts warning them about abnormal events that can be delivered to key team members
- Use CloudWatch dashboards to create various dashboards, that can be used to chart how you application is performing, and whether or not there are any issues with the application



## Optimising Data Operations

### Caching

- Caching can (and should be used where possible) speed up applications, by keeping a copy of frequently accessed data in memory
- An ElastiCache cluster consists of one or more nodes which are used to store frequently accessed data in memory
  - The number of nodes depends upon the demands your clients are likely to place on the application
  - ElastiCache clusters can be created using either one of two engines: Memcached or Redis
  - Memcached can be simpler to configure and deploy, is easily scalable, and because it works with multiple threads, can run faster, but isn't as flexible as it can only store BLOBs as key/value pairs
  - Redis, on the other hand, supports more complex data types such as strings, lists, and sets. Data can also be persisted to a disk, making it possible to snapshot data for later recovery
- Adding read-replicas to a database (RDS supports upto 5, and Aurora supports upto 15) can improve read times, as multiple reads can be distributed across multiple instances
- CloudFront is also a form of caching, and will reduce the latency between users and your origins, by caching data in between these points.

### Partitioning/Sharding

- Traffic volume on your RDS database can increase to the point that a single instance simply can’t handle it
- To solve this problem, you can use partitioning to split the data across multiple nodes. Additional application logic must be created, inorder to direct each data request to the correct partition

### Compression

- You can reduce the size of your data (and thus transfer times) by compressing the data before sending it to it's destination 



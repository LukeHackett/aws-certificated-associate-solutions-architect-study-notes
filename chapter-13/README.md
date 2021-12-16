# The Cost Optimisation Pillar

- The best protection from unpleasant surprises is by understanding how you’re billed for using AWS services, how you can deploy resources in the most cost-effective way, and how you can automate cost event–alerting protocols.

# Planning, Tracking, and Controlling Costs

- The place to find account-level financial information is the Billing and Cost Management Dashboard (found via the My Billing Dashboard link within the account drop down)

### AWS Budgets

- AWS can send ongoing billing reports to an S3 bucket you specify
- Budgets are meant to track your ongoing resource-related usage and costs and alert you if projected usage levels fall outside of a predefined threshold. Each account can have two free budgets
- AWS can email you billing alerts created through either Amazon CloudWatch Alarms or the newer AWS Budgets
- Cost allocations tags (Managed within the Cost Allocation Tags page) can be added to your resources, which can then be used as budget filters. The tags will let you limit your alerts to only the resources belonging to those tags, for example allowing you to track production costs, and not development or staging costs
  - Tags can take up to 24 hours before they appear in the Billing and Cost Management Dashboard
  - Tags can’t be applied to resources that were launched before the tags were created

### Monitoring Tools

- **Cost Explorer** allows you to drill down into very specific resources to deduce information about your usage, such as how much RI coverage you have
- **AWS Cost and Usage Reports** has similar functionality to the **Cost Explorer** however Cost and Usage Reports allow you to create reports that can be imported into Athena, Redshift or Quicksight, allowing you to perform custom analysis upon that data providing additional business intelligence. 
  - AWS Reports are meant for accounts that are so busy you struggle to keep track of many moving parts
  - Reports are written to an S3 bucket

### AWS Organisations

- You can consolidate the management of multiple AWS accounts using AWS Organisations
- This allows you to create resource isolation at an account level, while also being able to make a single payment all that will cover all accounts
- AWS Organisations lets you create new accounts and invite existing accounts into an organisation
- Organisation administrations can create global access policies much the same way as you might for individual users within a single account
- AWS Organisations allows you to:
  - Share resources through a unified AWS Single Sign-on configuration that applies global permissions to developers for all your accounts
  - Apply IAM rules globally through service control policies (SCPs)
  - Create and manage accounts — along with account users and groups — programmatically
  - Audit, monitor, and secure all your environments for compliance and functional purposes

### AWS Trusted Advisor

- The Trusted Advisor Dashboard will provide you an overview as to how closely you account configurations follow AWS best practises
- The compliance and health data is divided into five categories:
  - **Cost Optimisation** which checks to ensure there are no idle or underutilised resources
  - **Performance** which checks for configuration mismatches that might results in a performance impact such as excessive use of EBS magnetic volumes
  - **Security** checks for potentially vulnerable configurations such as public S3 bucket permissions
  - **Fault Tolerance** checks for appropriate replication and redundancy configurations to ensure you applications are fault tolerant
  - **Service Limits** checks for your usage of AWS services to make sure you’re not approaching default limits
- Sometimes these warnings can be ignored, such as an S3-hosted website may need to be public in order for it to serve to the internet

### Online Calculator Tools

- AWS provides two regularly updated calculators that can provide you estimates for any applications you wish to host on AWS: the **Simple Monthly Calculator** and the **AWS TCO Calculator**.
- The **Simple Monthly Calculator** lets you one or more select service resources and generate an estimated monthly bill. Each estimate can be exported as a CSV file. Costs are region specific, hence the price might be different for the same stack within different regions
- The **TOC Calculator** estimates the total cost of ownership (TCO) to help you make an apples-to-apples comparison of what a complex workload would cost on-premises as opposed to what it would cost on the AWS cloud



# Cost-Optimisation

### Maximising Server Density

- Matching your workload to the right instance type can make a big difference not only in performance but also in overall cost - for example a more powerful EC2 instance may be able to run more resource hungry workloads vs multiple smaller less powerful machines
- AWS Lambda also provides a kind of server density; it may not be your server resources you’re conserving by running Lambda functions, but you’re certainly getting a lot more value from your compute dollar by following the "serverless" model
- Amazon ECS and EKS, as well as Docker on EC2 instances will provide the most compute for money, as these technologies are designed to tightly pack virtual applications or microservices into every corner of your host hardware

### EC2 Reserved Instances

- A reserved instance is the right to use an EC2 instance at a discounted price, but committing to purchase compute at an partial or full upfront cost (which will be discounted)
- The Amazon EC2 Reserved Instance Marketplace, allows you to purchase (by taking over the time remaining on a reserved instance) an existing RI that would have been owned by other AWS users who have changed their usage plan
- An RI can only be used for the instance type that it was originally purchased for, unlike a convertible RI which allows you to exchange your instance later as long as the new instance has equal or greater value than the original
- Convertible RIs provide a discount, but not as much as a standard RI
- All RIs can be purchased using any one of three payment options: All Upfront (the lowest price), Partial Upfront, and No Upfront (billed hourly)

### Savings Plans

- Savings Plans are a lot like RIs in that you commit to a one to three-year period of constant service usage, but they offer significantly more flexibility
  - **Compute Savings Plans** offer up to 66% discounts over On Demand workloads for any EMR, ECS, EKS, or Fargate workload, and can be used to switch between any Region. Note: this does not include EC2 instances.
  - **EC2 Instance Plans** offer up to 72% discounts over On Demand workloads but can only be used within a single AWS region.

### EC2 Spot Instances

- EC2 spot instances — short-term “rentals” of EC2 instances — can offer very low prices. There are a number of considerations to take into account with Spot Instances:
  - **Spot Price** - the current going rate for spot instances of a given set of launch specifications (type, region, and profile values) - not prices can rise or fall without warning resulting in an instance shutting down
  - **Spot Instance Interruption** - what to do when the spot instance is interrupted: Terminate (permanently delete
    all associated resources and volumes), Stop (possible only when you’re using an EBS-backed AMI), and Hibernate
  - **Spot Instance Pool** - All the unused EC2 instances matching a particular set of launch specifications. Spot fleets are drawn from matching spot instance pools.
  - **Spot Fleet** - A group of spot instances that is launched to meet a spot fleet request - which can be made up of instances using multiple launch specifications
  - **Request Type** - the type of spot request you want AWS to deploy: Request (a one-time instance request), Request
    and Maintain (to maintain target capacity using a fleet), and Reserve For Duration (request an uninterrupted instance for between one and six hours)

- The Spot Advisor, which is available from the Spot Instances Dashboard, can helpfully recommend sample configuration profiles for a few common tasks
- Once you accept a recommendation, you can either customise it or send it right to the fleet configuration stage with a single click

### Auto Scaling

- An Auto Scaling configuration should also be set to scale your instances down as demand drops, as this will save a lot of money over the long term

### EBS Lifecycle Manager

- Backing up EBS volumes as snapshots helps to prevent data loss, however adding new snapshots every hour can add up to some intimidating storage costs
- The EBS Lifecyle Manager lets you create a rotation policy where you set the frequency at which new snapshots will be
  created and the maximum number of snapshots to retain

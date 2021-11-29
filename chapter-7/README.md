# CloudTrail, CloudWatch, and AWS Config

- CloudTrail, CloudWatch, and AWS Config are three services that can help you ensure the health, performance, and security of your AWS resources and applications
- CloudTrail, CloudWatch, and AWS Config are separate services that can be configured independently, but can also work together to provide a comprehensive monitoring solution for your AWS resources, applications and on-premises servers.



## CloudTrail

- An *event* is a record of an action that a principal performs against an AWS resource
- CloudTrail logs read and write actions against AWS services in your account, giving you a detailed record including the action, the resource affected and its region, who performed the action, and when

- CloudTrail logs both API and non-API actions:
  - API actions include launching an instance, creating an S3 bucket in S3, and creating a VPC.
  - Non-API actions include logging into the management console

- CloudTrail classifies events into *management events* and *data events*.

### Management Events

- A *management event* (also referred to as a *control plane operation*) includes operations that a principal executes (or attempts to execute) against an AWS resource
- Management events can be grouped into *write-only* or *read-only* events
  - **Write-Only Events** include operations that modify (or might modify) a resource such as create anew EC2 instance, or logging into the console as root or as a user (login creates a session)
  - **Read-Only Events** include operations that read resources, but can't make changes, such as returning a list of EC2 instances

### Data Events

- Data events track two types of data plane operations — S3 Object level activity and Lambda function executions — both of these tend to be high volume.
- CloudTrail distinguishes read-only and write-only events for S3 object-level operation, for example:
  - `GetObject` is an action to download an object from an S3 bucket and is read-only
  -  `DeleteObject` and `PutObject` are write-only events

### Event History

- CloudTrail logs 90 days of management events (not data events) and stores them in a viewable, searchable, and downloadable database called the *event history*
- CloudTrail creates a separate event history for each region containing only the activities that occurred in that region — however events for global services are included in the event history for every region

### Trails

- A *trail* records specified events (that you choose) and delivers them as CloudTrail log files to an S3 bucket of your choice — logs can be kept for any period of time (unlike the default event history)
- A trail can target a single or all regions, and if all regions are selected, AWS will automatically add any new regions it adds to your trail automatically
- Up to 5 trails can be created per region (this also includes trails across all regions)
- For data events, you are limited to selecting a total of 250 *individual objects* per trail, if you need more than this, you will need to select all objects, and filter out the ones that you are not interested in.
- It can take unto 10 minutes between the time CloudTrail logs an event and the time it writes a log file to the S3 bucket
- A log file contains many entries stored as a JSON document, with each record representing a single action against a resource. Each record will contain the following data:
  - **Event Time** the date and time of the action in UTC
  - **User Identity** details about the principal that initiated the request, such as the IAM User or IAM Role and it's ARN
  - **Event Source** the global endpoint of the service which the action was taken against, e.g `ec2.amazonaws.com`
  - **Event Name** name of the API operation, e.g. `RunInstances`
  - **AWS Region** the region the resource is located in, or `us-east-1` if the service is a global service
  - **Source IP address** the IP address of the requester

### Log File Integrity Validation

- CloudTrail provides a means to ensure that no log files were modified or deleted after creation
- Every hour, CloudTrail creates a separate file called a *digest file* that contains the cryptographic hashes of all log files delivered within the last hour
-  CloudTrail places this file in the same bucket as the log files but in a separate folder - allowing you to set permissions on the folder containing the digest file to protect it from deletion
- CloudTrail also cryptographically signs the digest file using a private key that varies by region and places the signature in the file’s S3 object metadata
- During quiet periods of no activity, CloudTrail also gives you assurance that no log files were delivered, as opposed to being delivered and then maliciously deleted
- You can validate the integrity of CloudTrail log and digest files by using the AWS CLI



## CloudWatch

- CloudWatch lets you collect, retrieve, and graph numeric performance metrics from AWS and non-AWS resources. 
- All AWS resources automatically send their metrics to CloudWatch, such as CPU and memory utilisation, EBS read/write IOPS, S3 bucket sizes
-  Custom metrics can be sent to CloudWatch from your applications and on-premises servers
- CloudWatch Alarms can send you a notification or take an action based on the value of those metrics. 
- CloudWatch Logs lets you collect, store, view, and search logs from AWS and non-AWS sources, whilst also being able to extract data from logs and create a metric.

### CloudWatch Metrics

- CloudWatch organises metrics into *namespaces* - AWS services are stored in AWS namespaces and use the format AWS/service, for example `AWS/EC2` is the namespace for EC2 metrics
- Namespaces help prevent metrics from being confused with similar names, for example WriteOps is used within `AWS/RDS` and `AWS/EBS`
- You can create custom namespaces for custom metrics — for example an apache web service might write under the `apache/http` namespace
- Metrics exist only in the region in which they were created
  - A metric functions as a variable and contains a time-ordered set of *data points* — each data point contains a timestamp, a value, and optionally a unit of measure.
  - Each metric is uniquely defined by a namespace, a name, and optionally a *dimension* - a dimension is a name/value pair that distinguishes metrics with the same name and namespace
  - Each EC2 instance will create a `CPUUtilization` metric in the `AWS/EC2` namespace for each instance, so ignorer to identify each instance, AWS assigns each metric a dimension named `InstanceId` with the value of the instance's ID.

#### Basic and Detailed Monitoring

- How frequently an AWS service sends metrics to CloudWatch depends on the monitoring type the service uses — *basic monitoring* and *detailed monitoring* is supported
- **Basic Monitoring** sends metrics to CloudWatch every five minutes (services such as EC2 provide basic monitoring), data is collected every minute, but only sends the five minute average to CloudWatch
- **Detailed Monitoring** sends metrics to CloudWatch every minute and is supported by more than 70 services, such as EC2, EBS, RDS, DynamoDB, ECS and Lambda
- How EC2 sends data points to CloudWatch depends upon the hypervisor:
  - Instances deployed on the Xen hypervisor (default) will publish metrics at the end of the five-minute interval
  - Instances deployed on the Nitro hypervisor will publish metrics every minute, and CloutWatch will treat the data point as a rolling average

#### Regular and High-Resolution Metrics

- The metrics generated by AWS services have a timestamp resolution of no less than one minute — these are known as *regular-resolution* metrics
- CloudWatch can store custom metrics with up to one-second resolution — these are known as *high-resolution metrics*
- You can create your own custom metrics using the `PutMetricData` API operation.


#### Expiration

- Metrics cannot be deleted in CloudWatch, but they will expire automatically depending on it's resolution — overtime CloudWatch aggregates higher-resolution metrics into lower-resolution metrics
- A high-resolution metric is stored for three hours, after this they are converted into one minute regular resolution metrics, with the high-resolution metric being expired during this conversion.
- After 15 days, the one minute regular resolution metric are converted into a 5 minute regular resolution metric (i.e. the average of a 5 minute block), which will be stored for 63 days.
- After 63 days, the 12 data points (one per 5 minutes) will be converted into an hourly data point, and stored for 15 months.
- After 15 months, the metrics are deleted. 

### Graphing Metrics

- CloudWatch can perform statistical analysis on data points over a period of time and graph the results as a time series — which can be used for showing trends.
- The following *statistics* are supported:
  - **Sum** - the total of all data points in a period
  - **Minimum** - the lowest data point in a period
  - **Maximum** - the highest data point in a period
  - **Average** - the average data point in a period
  - **Sample count** - the number of data points in a period
  - **Percentile** - the data point of the specified percentile, which can be up to two decimal places

- To graph a metric, you must specify the metric, the statistic, and the period, which can be from one second to 30 days, and the default is 60 seconds

### Metric Math

- CloudWatch lets you perform various mathematical functions against metrics and graph them as a new time series
- This capability is useful for when you need to combine multiple metrics into a single time series by using arithmetic functions, which include:
  - Addition, subtraction, multiplication, division, and exponentiation
  - Average
  - Max
  - Min
  - Sum
  - Standard deviation
- Statistical functions return a *scalar value*, not a time series, so they can’t be graphed. You must combine them with the METRICS function, which returns an array of time series of all selected metrics



### CloudWatch Logs

- CloudWatch Logs is a feature of CloudWatch that collects logs from AWS and non-AWS sources, stores them, and lets you search and even extract custom metrics from them
- Common uses include collecting application logs from an instance, logging Route 53 DNS queries and receiving CloudTrail logs.

#### Log Streams and Log Groups

- CloudWatch Logs stores *log events* that are records of activity recorded by an application or AWS resource
- Each log event must contain a timestamp and a UTF-8 encoded event message - you cannot store binary data in CloudWatch Logs
- Log events that originate from the same source (which maybe an application or AWS resource) are stored in a *log stream*.
- Log streams can be manually delete, but log events cannot be deleted
- Each log stream is organised into *log groups* - a stream can only exist in one log group - for example, you might stream logs from all instances in the same Auto Scaling group to the same log group
- You may define the retention settings for a log group, keeping logs for between one day to 10 years or indefinitely, which is the default setting. 
- A log group can be exported (manually) to an S3 bucket for archiving 

#### Metric Filters

- *Metric filters* can be used to extract data from log streams to create CloudWatch metrics
- All metric filters must be numeric, non-numeric strings such as an IP address, cannot be retrieved from a log. Alternatively, you can increment a value if a particular string has been matched - for example matching the number of times 404 appears in an Apache web server log to deduce how many 404 errors there have been
- Metric filters apply to entire log groups, and you can create a metric filter only after creating the group
- Metric filters are not retroactive and will not generate metrics based on log events that CloudWatch recorded before the filter’s creation

#### CloudWatch Agent

- The CloudWatch Agent is a command line–based program that collects logs from EC2 instances and on-premises servers running Linux or Windows operating systems
- Metrics generated by the agent are custom metrics and are stored in a custom namespace that you specify

#### Sending CloudTrail Logs to CloudWatch Logs

- CloudTrail to send a trail log to a CloudWatch Logs log stream - which allows you to search and extract metrics from your trail logs

- CloudTrail does not provide a way in which you can search through it's logs, and thus pushing them into CloudWatch Logs allows you to search through each JSON log, for example:

  ```json
  {$.eventSource = "signin.amazonaws.com" && $.responseElements.ConsoleLogin = "Failure" }
  ```

- CloudTrail does not send log events larger than 256 KB to CloudWatch Logs - and thus you must break up load requests if you want them to be available in CloudWatch logs



### CloudWatch Alarms

- A *CloudWatch alarm* watches over a single metric and performs an action based on a change in its value
- An action that can be taken includes sending an email notification, rebooting an instance or executing an Auto Scaling action

#### Data Point to Monitor

- Suppose you want to monitor the average of the AWS/EBS VolumeReadOps metric over a 15-minute period. 
  - The metric has a resolution of 5 minutes. 
  - You would choose Average for the statistic and 15 minutes for the period. 
  - Every 15 minutes CloudWatch would take three metric data points—one every 5 minutes—and would average them together to generate a single *data point to monitor*.

#### Threshold

- The threshold is the value the data point to monitor must meet or cross to indicate something is wrong. 
  - **Static Threshold** - a static threshold is defined by specifying a value and a condition, for example CPU Utilisation is greater than 90 percent
  - **Anomaly Detection** - is based on whether a metric falls outside of a range of values called a band. The size of the band is based upon the number of standard deviations - for example a threshold of 2 would mean alarms are triggered when the value is outside of two standard deviations from the average of values

#### Alarm States

- The period of time a data point to monitor must remain crossing the threshold to trigger an alarm state change depends on the *data points to alarm*
  - `ALARM` - The data points have crossed and remained past a defined threshold for a period of time
  - `OK` - The data points have not crossed and remained past a defined threshold for a period of time.
  - `INSUFFICIENT_DATA` - The alarm hasn’t collected enough data to determine whether the data points to alarm have crossed a defined threshold
- New alarms will always start out in an `INSUFFICIENT_DATA` state
- An alarm will only track only whether the data points to alarm have crossed and remained past a threshold for a period of time - it doesn't mean that if the alarm is OK that there are no issues.

#### Missing Data

- Missing data can occur during an evaluation period - this can happen when detaching an EBS volume or stopping an instance

- CloudWatch offers the following four options for how it evaluates periods with missing data
  - **As Missing** -  treats the data points as if they never occurred, CloudWatch will automatically ignore the missing data points. This is the default setting
  - **Not Breaching** - missing data points are treated as not breaching the threshold
  - **Breaching** - missing data points are treating as breaching the threshold, thus missing data points will be counted towards to data point threshold
  - **Ignore** - The alarm doesn’t change state until it receives the number of consecutive data points specified in the data points to alarm setting

#### Actions

- You can configure an alarm to take an action when it transitions to any given state - not just `ALARM`. An action can be one of the following:
  - **Notification Using SNS** - when the alarm triggers, it sends a notification to the configured topic
  - **Auto Scaling Action** - triggers an a simple Auto Scaling policy to add or remove instances (the policy must be defined before hand)
  - **EC2 Action** - can stop, terminate, reboot, or recover an instance in
     response to an alarm state change. Note: EC2 actions are available only if the metric you’re monitoring includes the InstanceId as a dimension, as the action you specify will take place against that instance



### Amazon EventBridge

- EventBridge monitors for and takes an action either based on specific events or on a schedule
- EventBridge differs from CloudWatch Alarms in that EventBridge takes some action based on specific events, not metric values

#### Event Buses

- EventBridge monitors event buses - every AWS account has one default event bus that receives events for all AWS services
- You can create a custom event bus to receive events from other sources, such as your applications or third-party services

#### Rules and Targets

- A rule defines the action to take in response to an event
- When an event matches a rule, you can route the event to a target that takes action in response to the event
- A rule can also invoke a target on a schedule, for example every hour



## AWS Config

- AWS Config tracks the configuration state of your AWS resources at a point in time, and can be used to see what a resource configuration looked like at some point in the past versus what it looks like now
- It can also show you how your resources are related to one another, highlighting if a change will affect other resources
- AWS Config deals with the *state* of a resource rather than actions that have been performed upon a resource (use CloudTrail or EventBridge for this)
- AWS Config can help you with the following objectives:
  - **Security** - AWS Config can notify you whenever a resource configuration changes, alerting you to potential breaches.
  - **Audit Reports** - You can provide a configuration snapshot report showing you how your resources were configured at any point in time
  - **Troubleshooting** - You can analyse how a resource was configured around the time a problem started, as well as using it to spot misconfigurations
  - **Change Management** - AWS Config lets you see how a potential change to one resource could impact another

### The Configuration Recorder

- The *configuration recorder* discovers your existing resources, records how they’re configured, monitors for changes, and tracks those changes over time.
- It will monitor all items in the region in which you configure it, but can also monitor global services such as IAM if required.
- There can only be one instance of the Configuration Recorder per region.

### Configuration Items

- The configuration recorder generates a *configuration item* for each resource it monitors.
- The configuration item contains the specific settings for that resource at a point in time, as well as the resource type, its ARN, when it was created and includes the resource’s relationships to other resources
- AWS Config maintains configuration items for every resource it tracks, even after the resource is deleted
- Configuration items are stored internally in AWS Config, and you can’t delete them manually

### Configuration History

- A configuration history is a collection of configuration items for a given resource over time
- A configuration history includes details about the resources, such as when it was created, how it was configured at different points in time, and when it was deleted (if applicable), as well as any related API logged by CloudTrail
- Every six hours in which a change occurs to a resource, AWS Config delivers a configuration history file to an S3 bucket that you specify (known as the *delivery channel*)
- Configuration history files are grouped by resource type, and each file is timestamped and kept in separate folders by date
- An SNS topic can be added to the delivery channel so that you are notified immediately whenever there's a change to a resource

### Configuration Snapshots

- A configuration snapshot is a collection of all configuration items from a given point in time — it works like a configuration backup for all monitored resources in your account
- AWS Config deliver a configuration snapshot to the bucket defined in your delivery channel - this cannot be setup within the console only via the CLI - you must specify JSON file that contains the delivery channel name, the s3 bucket name and the frequency of the snapshot data.
- The delivery frequency can be every hour or every 24, 12, 6, or 3 hours

### Monitoring Changes

- It is not possible to delete configuration items manually, you can configure AWS Config to keep configuration items between 30 days and 7 years, with 7 years being the default
- Note: the retention period does not apply to the configuration history and the configuration snapshot files stored on S3

#### Starting and Stopping the Configuration Recorder

- Starting and stopping the configuration recorder can be done at any time using the web console or the CLI
- When the configuration recorder is stopped, it doesn’t monitor or record changes, but it will retain existing configuration items

#### Recording Software Inventory

- AWS Config can record software inventory changes on EC2 instances and on-premises servers, which includes:
  - Applications
  - AWS components such as the CLI and SDKs
  - The name and version of the operating system 
  - IP address, gateway, and subnet mask 
  - Firewall configuration
  - Windows updates

- To have AWS Config track these changes, you must enable inventory collection for the server using the AWS Systems Manager

#### Managed and Custom Rules

- AWS Config lets you specify rules to define the optimal baseline configuration for your resources — for example you may want to verify that CloudTrail is enabled or that all EBS volumes are encrypted
- If any resources are noncompliant, AWS Config flags them and generates an SNS notification
- When you activate a rule, AWS Config immediately checks monitored resources against the rules to determine whether they’re compliant, and can be reevaluated every 3, 6, 12, or 24 hours depending upon what has been configured



# The Operational Excellence Pillar
- Operations include the ongoing activities required to keep your applications running smoothly and securely on AWS
- Achieving operational excellence requires automating some or all of the tasks that support your organisation's workloads, goals, and compliance requirements



## CloudFormation

- The resources defined by a CloudFormation template compose a *stack*
- Each CloudFormation template will have a number of resources that make up the stack, each being given a *logical ID* (sometimes known as the *logical name*), e.g. `MyLambdaFunction` or `PublicVPC`.
- CloudFormation can also read templates locally or from an S3 bucket
- Optional *parameters* may be defined and supplied to the CloudFormation template, allowing  you to pass custom values to the stack, rather than having to create multiple files
- Stacks can be deleted from the CLI or via the web console - if termination protection is not enabled on the stack, CloudFormation will immediately delete the stack and all associated resources
- Stacks can (and should) be split across multiple files, based upon life cycle and ownership - its common to pass values between each stack, which can be done via using nested stacks and exporting stack output values
  - A **Nested Stack** is one that sits within a parent stack. Each nested stack can reference values within other sibling stacks by getting an output attribute via the `Fn:GetAtt` intrinsic function
  - Sharing information with stacks outside of a nested hierarchy requires the use of the **stack output values**. Any other stack within the same account and region can import the values, via the `Fn::ImportValue` intrinsic function

- Stacks can updated using a **direct update** or via a **change set**:
  - Performing a **direct update** will involve uploading the updated template, and supplying any template parameters (if required), and CloudFormation will deploy the changes immediately, modifying the resources that changed in the template.
  - A **change set** will help you understand exactly what is going to be changed (and how it will be changed). CloudFormation will display a list of every resource it will add, remove, or modify. You can then choose to execute the change set to make the changes immediately, delete the change set, or do nothing.
- CloudFormation will update a resource depending upon the chosen update behaviour:
  - **Update with No Interruption** - the resource is updated with no interruption and without changing it's ID, examples include IAM instance profile changes as it doesn't require an interruption to the EC2 instance
  - **Update with Some Interruption** - the resource encounters a momentary interruption but retains it's physical ID, for example changing the instance type of an EBS-backed EC2 instance
  - **Replacement** - CloudFormation creates a new resource with a new physical ID, then it will point the dependent resources to the new resource, deleting the original resource, examples include moving an EC2 instance into a different availability zone
- To prevent specific resources in a stack from being modified by a stack update, you can create a stack policy when you create the stack. A stack policy follows the same format as other resource policies and consists of the same `Effect`, `Action`, `Principal`, `Resource` and `Condition` elements:
  - **Action** - must be one of the following:
    - `Update:Modify` - allows updates to the specific resource only if the update will modify and not replace or delete the resource
    - `Update:Replace` - allows updates to the specific resource only if the update will replace the resource
    - `Update:Delete` - allows updates to the specific resource only if the update will delete the resource
    - `Update:*` - allows all update actions
  - **Principal** - must always be the wildcard (*), it is not possible to specify any other principal
  - **Resource** - specifies the logical ID of the resource to which the policy applies. You must prefix it with the text `LogicalResourceId/`, for example `LogicalResourceId/PublicVPC` would only apply to the `PublicVPC` resource within the stack
  - **Condition** - specifies the resource types, such as `AWS::EC2::VPC`. Wildcards are supported, as well as wildcards within a service type, such as `AWS::EC2::*`

- You can temporarily override a stack policy when doing a direct update



## CodeCommit

- The AWS CodeCommit service hosts private Git-based repositories for version-controlled files, including code, documents, and binary files
- A CodeCommit repository can be created using the AWS Management Console or the AWS CLI
- Files can be added to a CodeCommit repository via:
  - Uploading files via the CodeCommit management console
  - Use to `git` to commit and push files to the repository
- CodeCommit uses IAM policies to control access to repositories. AWS provides three managed policies:
  - **AWSCodeCommitFullAccess** - provides full access to CodeCommit, and is usually given to administrators
  - **AWSCodeCommitPowerUser** - provides near full access, but doesn't allow deleting repositories, and is usually given to users who need to read and write access
  - **AWSCodeCommitReadOnly** - provides read-only access
- Only IAM principals can interact with a CodeCommit repository, and access is provided one of:
  - A git username and password generated from within the IAM Console
  - An ssh key which is uploaded within the IAM Console



## CodeDeploy

- CodeDeploy is a service that can deploy applications to EC2, on-premises instances or Lambda functions
- The CodeDeploy agent is a service that runs on your Linux or Windows instances and performs the hands-on work of deploying the application onto an instance

### Deployments

- To deploy an application using CodeDeploy, you must create a *deployment* that defines the compute platform, which can be EC2/on-premises or Lambda, and the location of the application source files
- CodeDeploy currently supports only deployments from S3 or GitHub
- CodeDeploy does not deploy applications automatically - to do this, you need to use CodePipeline
- A *deployment group* is used to define which instances CodeDeploy will deploy your application to, and can be based on:
  - An EC2 Auto Scaling group
  - EC2 instance tags
  - On-premises instance tags
- When creating a deployment group for EC2 or on-premises instances, you must specify a *deployment type*
  - **In-Place Deployment**
    - The application is deployed to the existing EC2 instances, by stopping and upgrading the application, and then restarting it
    - If the EC2 instances are registered to an ELB, CodeDeploy will automatically remove it from the ELB target while the upgrade is happening
  - **Blue/Green Deployment**
    - The application is deployed to new instances, and traffic is shifted over to the new instances once the deployment is completed
    - CodeDeploy will not modify the minimum, maximum, or desired capacity settings for an Auto Scaling group (if being used)
    - You can choose to terminate or keep the original instances if you need them for testing or forensic analysis

- When creating your deployment group, you must also select a *deployment configuration*, which defines the number of instances CodeDeploy will simultaneously deploys to, as well as how many instances the deployment must succeed on for the entire deployment to be considered successful
  - **OneAtATime** - CodeDeploy will deploy to one instance at a time before moving on, with the overall deployment being marked as a success if all the but the last instance has succeeded. For blue/green deployments, CodeDeploy will reroute the traffic to each instance as the deployment succeeds on the first instance
  - **HalfAtATime** - CodeDeploy will deploy up to half of the instances in the deployment group before moving on to the remaining instances, with the deployment only succeeding if the deployment was successful to at least half the instances. For blue/green deployments, CodeDeploy must be able to reroute traffic to at least half of the new instances for the entire deployment to succeed.
  - **AllAtOnce** - CodeDeploy simultaneously deploys the application to as many instances as possible, with the deployment being regarded a success if the application is deployed to at least one instance. For green/blue deployments, the deployment will success if CodeDeploy is able to reroute traffic to at least one new instance

### Lifecycle Events

- An instance deployment is divided into lifecycle events, to which you can have the agent execute a *lifecycle event hook* (a script you develop). The supported lifecycle events are as follows:
  - **ApplicationStop** - can be used to gracefully stop an application, or perform clean up tasks as part of stopping the application
  - **BeforeInstall** - occurs after the agent has copied the application files into a temp location within the instance. This stage can be used to perform some additional actions before moving them to the final location, such as decrypting files
  - **AfterInstall** - after the agent copies your application files to their final destination, this hook performs further needed tasks, such as setting file permissions
  - **ApplicationStart** - you use this hook to start your application, such as starting an apache web server
  - **ValidateService** - you can check that the application is working as expected, such as generating logs files, or has a connection to the database
  - **BeforeBlockTraffic** - with an in-place deployment using an elastic load balancer, this hook occurs first, before the instance is unregistered
  - **AfterBlockTraffic** - occurs after the instance is unregistered from an elastic load balancer, and allows for traffic to be gracefully served before updating the instance
  - **BeforeAllowTraffic** - occurs after the application is installed and validated, and allows you to perform any tasks before traffic is served to the instance, such as warming up a cache (only supported for deployments with an ELB)
  - **AfterAllowTraffic** - the final event for deployments using an elastic load balance

### The Application Specification File

- The application specification (AppSpec) file defines where the agent should copy the application files onto your instance and what scripts it should run during the deployment process. It consists of the following five sections:
  - **Version** - only 0.0 is supported
  - **OS** - specifies the OS the CodeDeploy agent is running on
  - **Files** - one or more source and destination pairs used to copy from source to destination
  - **Permissions** - optionally sets ownership, group membership file permissions and Security-Enhanced Linux labels. Only works with Amazon Linux, Ubuntu and Redhat instances.
  - **Hooks** - define the scripts that the agent should run at each lifecycle event, which contain the location of the script, how long the script is expected to run for (timeout) and which user to run as

### Triggers and Alarms

- You can optionally set up triggers to generate an SNS notification for certain deployment and instance events, such as when a deployment succeeds or fails
- A deployment group may monitor unto 10 CloudWatch alarms, and if an alarm exceeds or falls below a threshold, the deployment will stop

### Rollbacks

-  CodeDeploy can optionally be configured to *roll back* to the last successful revision of an application if the deployment fails or if a CloudWatch alarm is triggered during deployment



## CodePipeline

- Continuous integration is a method whereby developers use a version control system to regularly submit or check in their changes to a common repository
- Continuous delivery incorporates elements of the CI process but also deploys the application to production
- CodePipeline lets you automate the different *stages* of your software development and release process
- Every CodeDeploy pipeline must include *at least two stages* and can have *up to 10 stages*
- Each stage must have one or more *actions* (up to a total of 20), and can be run sequentially or in parallel
- Each action must be one of the following types:
  - **Source**
    - Specifies the source of your application files, and must be the first stage of a pipeline
    - Valid providers are CodeCommit, S3 or Github
  - **Build**
    - Builds the source code (not all languages require to be compiled, but this stage can still be used for source code analysis)
    - This action supports CodeBuild, as well as third party providers, such as Jenkins and TeamCity
  - **Test**
    - Tests the source code, an can optionally integrate with other testing services if required
  - **Approval**
    - Only supports manual approvals, which will make the pipeline wait, until someone manually approves it into the next stage.
    - Particularly useful for when deploying to production
  - **Deploy**
    - Deploys the application by integrating with CodeDeploy, CloudFormation, Elastic Container Service, Elastic Beanstalk, OpsWorks Stacks
  - **Invoke**
    - Allows the execution of a Lambda function as part of the pipeline, for example you can create a Lambda function to perform an EBS snapshot before pushing to production

### Artifacts

- When you create a pipeline, you must specify an S3 bucket to store the files used during different stages of the pipeline
-  CodePipeline compresses these files into a zip file called an *artifact*.
- The *artifact* can be supplied as an input into the next action, for example after running the build action, a "build" artifact is created, which can be passed to the test action.
- This process continues throughout the pipeline - and is the reason why you must specify an IAM service role for CodePipeline to assume (with S3 read/write permissions) when creating a pipeline



## AWS Systems Manager

- AWS Systems Manager lets you automatically or manually perform *actions* against your AWS resources and on-premises servers

### Actions

- Actions let you automatically or manually perform actions against your AWS resources, either individually or in bulk
- Each action must be defined within documents (Automation, Command or Policy)

#### Automation

- Allows you to perform actions against your AWS resources in bulk, such as restarting multiple EC2 instances or patching AMIs
- Tasks can be performed in one fell swoop or it can perform on step at a time, as well as allowing you to specify how may resources as a percentage to target at once

#### Run Command
- Allows you to execute scripts on an EC2 instance, when normally you'd have to login an execute it (or use a third party tool)
- This is achieved by installing the Systems Manager agent on the EC2 instance
-  Other documents can be used such as restarting a Windows service or installing an application
- Instances can be targeted by tags or by selecting them all at once

#### Session Manager

- Session Manager lets you achieve interactive Bash and PowerShell access to your instances, without having to open inbound ports on a security group or a network ACL or even having your instances in a public subnet
- Connections made via Session Manager are secured using TLS 1.2
- Session Manager can keep a log of all logins in CloudTrail and store a record of commands run within a session in an S3 bucket

#### Patch Manager

- Patch Manager helps you automate the patching of your Linux and Windows instances
- Individual EC2 instance can be selected to be patched, via tags or a *patch group*.
- A *patch group* is a collection of instances with the tag `Patch Group`, for example a group for web servers might be `Patch Group => web-servers`.
- Patch Manager uses *patch baselines* to define which available patches to install, as well as whether the patches will be installed automatically or require approval.
- Patch baselines are classified as security-related, critical, important, or required, with all operating systems (except Ubuntu) automatically approving these patches after seven days — this is known as an auto-approval delay
- Custom baselines can be created which allows you to define the severity level of patches to install, and an auto-approval delay
- If a patch is approved, it will be installed during a maintenance window that you specify, or it can patch the instances immediately

#### State Manager

- State Manager is a configuration management tool that ensures your instances have the software you want them to have and are configured in the way you define
- State Manager can automatically run command and policy documents against your instances, either one time only or on a schedule, such as ensuring anti-virus software is running on each instance
- State Manager can also gather software inventory install on EC2 instances, as well as collecting network configurations; file information; CPU information; and, for Windows, registry values

### Insights

- Insights aggregate health, compliance, and operational details about your AWS resources into a single area of AWS Systems Manager
- Insights can be categorised as *AWS resource groups*, allowing you to see all the resources related to a particular application, such as EC2 instance, S3 buckets, EBS volumes etc

#### Built-In Insights

- Built-in insights are provided by default, and include the following:

  - **AWS Config Compliance** - shows the total number of resources in a resource group that are compliant or noncompliant with AWS Config rules, as well as compliance by resource

  - **CloudTrail Event** - displays each resource in the group, the resource type, and the last event that CloudTrail recorded against the resource.

  - **Personal Health Dashboard** - contains alerts when AWS experiences an issue that may impact your resources, as well as showing the number of events that AWS resolved within the last 24 hours

  - **Trusted Advisor Recommendations** - checks your AWS environment for optimisations and recommendations for cost optimisation, performance, security, and fault tolerance. It will also show you when you’ve exceeded 80 percent of your limit for a service. Only Business and Enterprise customer get access to all the Trusted Advisor Recommendations

#### Inventory Manager

- The Inventory Manager collects data from your instances, including operating system and application versions, such as:
  - Operating system name and version
  - Applications and filenames, versions, and sizes
  - Network configuration, including IP and MAC addresses 
  - Windows updates, roles, services, and registry values 
  - CPU model, cores, and speed

- You choose which instances to collect data from by using tags or by choosing all instances in the account (known as *global inventory association*)
- Inventory collection occurs at least every 30 minutes

#### Compliance

- Compliance insights show how the patch and association status of your instances stacks up against the rules you’ve configured
- Patch compliance shows the number of instances that have the patches in their configured baseline
- Association compliance shows the number of instances that have had an association successfully executed against them



## AWS Landing Zone

- AWS Landing Zone can automatically set up a multi-account environment according to AWS best practices, which helps to reduce the amount of manual setup that may be required
- Landing Zone begins with four core accounts, each containing different services and performing different roles:
  - **AWS Organizations Account** - an account that allows you to create and manage other member accounts
  - **Shared Services Account** - an account that contains a shared services VPC and is designed for hosting services that other accounts can access
  - **Log Archive Account** - an account that CloudTrail and AWS Config log files
  - **Security Account** - an account that contains GuardDuty, cross-account roles, and SNS topics for security notifications

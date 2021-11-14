# AWS Storage



## Amazon S3

- Simple Storage Service (S3) lets you store and retrieve unlimited amounts of data. Because it’s part of the AWS ecosystem, data in S3 is available to other AWS services, including EC2, making it easy to keep storage and compute together for optimal performance.
- Many AWS services use S3 to store logs, retrieve data for processing, or to host static websites.

### Objects and Buckets

- S3 stores files (or *objects*) on disks in AWS data centres, which is different from EBS which stores blocks of raw data.
- S3 stores objects in a container called a *bucket* that’s essentially a flat file system (folders can be created by including a forward slash in the object name). Each bucket must have a name that is between 3 and 63 characters long, and the name *must be globally unique* across AWS (this is to ensure urls are mapped to the correct object and bucket).
- Each file's name is known as a key, and can be up to 1024 bytes long, and can consist of alphanumeric characters and some special characters, including `! - _ . * ( )`
- You can store any type of file up to 5TB in size, however Individual uploads can be no larger than 5 GB. 
- To reduce the risk of data loss or aborted uploads, AWS recommends that you use a feature called Multipart Upload for any object larger than 100 MB.
- Transfer Acceleration will allow for faster uploads to the bucket, by routing the upload through geographically nearby AWS edge locations and, from there, are routed using Amazon’s internal network. In order to use Transfer Acceleration it needs to be enabled on the bucket, and you must use a special endpoint domain name, such as bucketname.s3-accelerate.amazonaws.com for your transfers.
- Data is technically only stored in one region, but can be replicated across multiple regions.
- You’re not charged for uploading data to S3, but you may be charged when you download data, as well as being charged based upon how much data you store and the storage class you use.

### S3 Storage Classes

- Objects that need to remain intact and free from inadvertent deletion or corruption are said to need high *durability*, which is the percent likelihood that an object will not be lost over the course of a year. The greater the durability of the storage medium, the less likely you are to lose an object.
- *Availability* is the percent of time an object will be available for retrieval. For instance, a patient’s medical records may need to be available around the clock, 365 days a year. Such records would need a high degree of both durability and availability.
- S3 replicates data across multiple locations, alas, there might be brief delays while updates to existing objects propagate across the system (usually no more than two seconds). Because there isn’t the risk of corruption when creating new objects, S3 provides read-after-write consistency for creation (PUT) operations.
- The level of durability and availability of an object depends on its storage class. S3 offers six different storage classes at different price points. 

| Storage Class              | Durability    | Availability | Availability Zones | Recommended Access Level | Example Pricing <br />(per GB/month)                         |
| -------------------------- | ------------- | ------------ | ------------------ | ------------------------ | ------------------------------------------------------------ |
| `STANDARD`                 | 99.999999999% (11 nines) | 99.99%       | \>2                | Frequent                 | $0.023                                                       |
| `STANDARD_IA`              | 99.999999999% (11 nines) | 99.9%        | \>2                | Infrequent               | $0.0125                                                      |
| `INTELLIGENT_TIERING`      | 99.999999999% (11 nines) | 99.9%        | \>2                | Frequent and Infrequent  | Frequent access tier: $0.023<br />Infrequent access tier: $0.0125 |
| `ONEZONE_IA`               | 99.999999999%  (11 nines) | 99.5%        | 1                  | Infrequent               | $0.01                                                        |
| `GLACIER`                  | 99.999999999% (11 nines) | Varies       | \>2                | Infrequent               | $0.004                                                       |
| `REDUCED_REDUNDANCY (RRS)` | 99.99%        | 99.99%       | \>2                | Frequent                 | $0.024                                                       |

#### Standard Storage Class
- This is the default storage class.
- It offers the highest levels of durability and availability.
- Objects are always replicated across at least three Availability Zones in a region.

#### Reduced Redundancy Storage Class
- This storage class is meant for data that can be easily replaced.
- It offers the lowest levels of durability and highest levels of availability.
- AWS recommends against using this storage class (but keeps it available for people who have processes that still depend on it).

#### Standard IA Storage Class
- This storage class is designed for important data that cannot be re-created.
- Objects are stored in multiple Availability Zones and have an availability of 99.9 percent.
- Offer millisecond-latency access and high durability but the lowest availability of all the classes.

#### OneZone IA Storage Class
- Objects stored using this storage class are kept in only one Availability Zone and consequently have the lowest availability of all the classes.
- The destruction of an Availability Zone could result in the loss of objects stored in that zone.
- Use this class only for data that you can re-create or have replicated elsewhere.
- Offer millisecond-latency access and high durability but the lowest availability of all the classes.

#### Glacier Storage Class
- The `GLACIER` class is designed for long-term archiving of objects that rarely need to be retrieved.
- Unlike the other storage classes, you can’t retrieve an object in real time, you must make a request for the data, the time in which this takes depends on the retrieval option you have selected — this ranges from 1 minute to 12 hours.

#### Intelligent Tiering Storage Class
- This storage class automatically moves objects to the most cost-effective storage tier based on past access patterns.
- An object that hasn’t been accessed for 30 consecutive days is moved to the lower-cost infrequent access tier.
- Once the object is accessed, it’s moved back to the frequent access tier.
- In addition to storage pricing, you’re charged a monthly monitoring and automation fee.

### Encryption

- AWS does not encrypt data uploads by default. Unless it’s intended to be publicly available — perhaps as part of a website — data stored on S3 should always be encrypted.
- S3 gives you the two options for encrypting objects — *server-side encryption* or *client-side encryption*.

#### Server-Side Encryption

- When you create an object, S3 encrypts the object and saves only the encrypted content.
- When you retrieve the object, S3 decrypts it and delivers the unencrypted object.
- One of three encryption options are supported:
  - **Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3)** - data is encrypted using AWS's own enterprise-standard keys
  - **Server-Side Encryption with AWS KMS-Managed Keys (SSE-KMS)** - data is encrypted using the same method as SSE-SE, but also adds an *envelope key* as well as a full audit trail for tracking key usage. SSE-KMS also supports using your own keys, through the AWS KMS service.
  - **Server-Side Encryption with Customer-Provided Keys (SSE-C)** - allows you to provide your own keys for S3 to apply to its encryption.

#### Client-Side Encryption

- Data is encrypted before it’s transferred to S3 using an AWS KMS–Managed Customer Master Key (CMK).
- The CMK produces a unique key for each object before it’s uploaded or alternatively, you can also use a Client-Side Master Key, which you provide through the Amazon S3 encryption client.

### Logging

- Tracking S3 events to log files is disabled by default — S3 buckets can see a lot of activity, and thus may greatly increase costs

- When you logging is enabled you’ll need to specify both a source bucket (the bucket whose activity you’re tracking) and a target bucket (the bucket to which you’d like the logs saved).
- You can also specify delimiters and prefixes (such as the creation date or time) to make it easier to identify and organise logs from multiple source buckets that are saved to a single target bucket.
- S3 generated logs, which sometimes appear only after a short delay, will contain basic operation details, including the following:
  - The account and IP address of the requester
  - The source bucket name
  - The action that was requested ( GET , PUT , POST , DELETE , etc.)
  - The time the request was issued
  - The response status (including error code)

### Lifecycle Management

- S3 can store unlimited amounts of data, and thus it’s possible to run up quite a bill if you continually upload objects but never delete any.
- Object life cycle configurations can help you control costs by automatically moving objects to different storage classes or deleting them after a time.
- Object life cycle configuration rules are applied to a bucket and consist of *one or both* of the following types of actions:
  - **Transition actions**
    - Transition actions move objects to a different storage class once they’ve reached a certain age
  - **Expiration actions**
    - These can automatically delete objects after they reach a certain age.
    - If you have versioning enabled on a bucket, you can create expiration actions to delete object versions of a certain age.

### Versioning

- Versioning is disabled by default when you create a bucket.
- You must explicitly enable versioning on a bucket to use it, and it applies to all objects in the bucket - once enabled it cannot be disabled (only paused).
- There’s no limit to the number of versions of an object you can store.
- You can delete versions manually or automatically using object life cycle configurations.

### Accessing S3 Objects

#### Access Control
- By default, S3 buckets and objects will be fully accessible to your account but to no other AWS accounts or external visitors.
- You can open up access at the bucket and object levels using access control list (ACL) rules, finer-grained S3 bucket policies, or Identity and Access Management (IAM) policies.
- ACLs are leftovers from before AWS created IAM, alas Amazon recommends applying S3 bucket policies or IAM policies instead of ACLs.
- S3 bucket policies will make sense for cases where you want to control access to a single S3 bucket for multiple external accounts and users. On the other hand, IAM policies are used when you’re trying to control the way individual users and roles access multiple resources, including S3.

#### Pre-signed URLs

- If you want to provide temporary access to an object that’s otherwise private, you can generate a presigned URL. The URL will be usable for a specified period of time, after which it will become invalid.
- The following AWS CLI command will return a URL that includes the required authentication string. The authentication will become invalid after 600 seconds. The default expiration value is 3,600 seconds (one hour).


```shell
$ aws s3 presign s3://MyBucketName/PrivateObject --expires-in 600
```


#### Static Website Hosting

- S3 buckets can be used to host the HTML files for entire static websites.
- If you want requests for a DNS domain name (like mysite.com ) routed to your static site, you can use Amazon Route 53 to associate your bucket’s endpoint with any registered name.
- You can also get a free SSL/TLS certificate to encrypt your site by requesting a certificate from AWS Certificate Manager (ACM) and importing it into a CloudFront distribution that specifies your S3 bucket as its origin.
- AWS provides a different way to access data stored on either S3 Select or Glacier Select - such as finding records within a given CSV file - without having to download, and parse the entire file.



## Amazon S3 Glacier

- S3 Glacier offers long-term archiving of infrequently accessed data, such as backups that must be retained for many years.
- With Glacier, you store one or more files in an archive, which is a block of information. Although you can store a single file in an archive, the more common approach is to combine multiple files into a .zip or .tar file and upload that as an archive
- An archive can be anywhere from 1 byte to 40 TB.
- Glacier stores archives in a vault, which is a region-specific container (much like an S3 bucket) that stores archives. Vault names must be unique within a region but don’t have to be globally unique
- Glacier archives are encrypted by default

- You can create and delete vaults using the Glacier service console. But to uDeep Archive Standardpload, download, or delete archives, you must use the AWS command line interface.
- There are currently two Glacier storage tiers: **Standard** and **Deep Archive**, with the latter costing less but will require longer waits for data retrieval.
- Downloading an archive from Glacier is a two-step process that requires initiating a retrieval job and then downloading your data once the job is complete.
- The length of time it takes to complete a job depends on the retrieval option you choose.
  - **Glacier Expedited** - retrievals usually complete within 1 to 5 minutes (unless there is high demand), although archive larger than 250MB may take longer, expedited retrievals are the most expensive.
  - **Glacier Standard** - retrievals usually complete within 3 to 5 hours, and is the default retrieval option.
  - **Glacier Bulk** - retrievals usually complete within 5 to 12 hours.
  - **Deep Archive Standard** - retrievals usually complete within 12 hours.



## Amazon Elastic File System (EFS)

- EFS provides automatically scalable and shareable file storage to be accessed from *Linux* instances.
- EFS-based files are designed to be accessed fromwithin a virtual private cloud (VPC) via Network File System (NFS) mounts on EC2 Linux instances or from your on-premises servers through AWS Direct Connect connections. 
- The goal is to make it easy to enable secure, low-latency, and durable file sharing among *multiple instances.*



## Amazon FSx

- Amazon FSx comes in two flavors: **FSx for Lustre** and **Amazon FSx for Windows File Server**.

- Lustre is an open source distributed filesystem built to give Linux clusters access to high-performance filesystems for use in compute-intensive operations. Amazon’s FSx service brings Lustre capabilities to your AWS infrastructure.
- FSx for Windows File Server, provides an elastic filesystem for Windows servers rather than Linux. FSx for Windows
  File Server integrates operations with Server Message Block (SMB), NTFS, and Microsoft Active Directory.



## AWS Storage Gateway

- AWS Storage Gateway provides software gateway appliances with multiple virtual connectivity interfaces.
- Local devices can connect to the appliance as though it’s a physical backup device like a tape drive, and the data itself is saved to AWS platforms like S3 and EBS. 
- Data can be maintained in a local cache to make it locally available.



## AWS Snowball

- AWS Snowball is a hardware storage appliance designed to physically move massive amounts of data to or from S3, particularly when transferring the data over a network would take days or weeks.
- For example, suppose you want to migrate a 40 TB database to AWS. Such a transfer even over a blazing-fast 1 Gbps connection would still take more than 4 days!
- AWS will send you (for a fee) a 256-bit, encrypted Snowball storage device onto which you’ll copy your data. You then ship the device back to Amazon, where its data will be uploaded to your S3 bucket(s).
- You’re not charged any transfer fees for importing files into S3.



## AWS DataSync

- DataSync specializes in moving on-premises data stores into your AWS account with a minimum of fuss. 
- It works over your regular Internet connection (larger dataset maybe better off with AWS Snowball).
- Using DataSync, you can drop your data into any service within your AWS account. That means you can do the following:
  - Quickly and securely move old data out of your expensive data center into cheaper S3 or Glacier storage.
  - Transfer data sets directly into S3, EFS, or FSx, where it can be processed and analyzed by your EC2 instances.
  - Apply the power of any AWS service to any class of data as part of an easy-to-configure automated system.

- DataSync can handle transfer rates of up to 10 Gbps (assuming your infrastructure has that capacity) and offers both encryption and data validation.


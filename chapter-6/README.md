# Authentication and Authorisation
- On AWS, authentication and authorisation are primarily handled by Identity and Access Management (IAM)
- An IAM identity (some times described a *principals*), represents an AWS user or a role
- Roles are identities that can be temporarily assigned to an application, service, user, or group
- Identities can also be federated — that is, users or applications without AWS accounts can be authenticated and given temporary access to AWS resources using an external service such as Kerberos, Microsoft Active Directory, or LDAP
- Identities are controlled by attaching policies that precisely define the way they’ll be able to interact with all the resources in your AWS account



## IAM Identities

- The root user is a predefined account that AWS provides, that has full rights over all the services and resources associated with your account
- Due to the nature of the root account, AWS suggests that you heavily protect your root account and delegate specific powers for day-to-day operations to other users

### Protecting the Root Account

- The best way to protect your root account is to lock it down by doing the following:
  - Delete any access keys associated with root
  - Assign a long and complex password and store it in a secure password vault
  - Enable multi-factor authentication (MFA) for the root account
  - Wherever possible, don’t use root to perform administration operations (delegate to other user accounts using policies)

- Amazon provides a list of recommended actions in the Security Status section of a typical account’s IAM home page for tasks that the user should perform in order to make their account secure
- **Note:** the `AdministratorAccess` policy does not have complete control over the account as the root account does, for example you'll need to be root in order to create account-wide budgets

### IAM Policies

- An IAM policy is a document that identifies one or more actions as they relate to one or more AWS resources
- The policy document also determines the effect permitted by the action on the resource - either Allow or Deny
- For example, a policy might, *allow* (the effect) the *creation of buckets* (the action) within *S3* (the resource)
- You can use a preset policy or you can create your own using the tools available through the Create policy page in the Dashboard or by manually crafting your own using JSON
- Any action that’s not explicitly allowed by a policy will be denied
- If two policies are associated with a single identity conflict, AWS will resolve the conflict by denying the action

### User Accounts

- A password policy can be assigned to any user, which can be used to enforce minimum lengths, complexity along with how frequently the password should be altered.
- Users can open the My Security Credentials page, and will be able to perform the following actions:
  - Update their password
  - Activating (or manage) their MFA
  - Generating or deleting access keys for managing your AWS resources via the CLI/SDK
  - Generating key pairs for authenticating signed URLs for your Amazon CloudFront distributions
  - Generating X.509 certificates to encrypt SOAP requests to AWS service that support this
  - Retrieving your 12-digit AWS Account ID


### Access Keys

- Access keys provide authentication for programmatic or CLI-based access.
- Access keys should be audited on a regular basis, ensuring that any keys that are not being used are deactivated, or better removed all together

#### Key Rotation

- Best practices require that you regularly retire older access keys because the longer a key has been in use, the greater the chance that it’s been compromised
- Key rotation is automated for IAM roles used by EC2 resources to access other AWS services
- You can enforce key rotation by including rotation in the password policy associated with your IAM user accounts

### Groups

- A Group can be used to administrate the permissions associated with multiple users (much in the same way Linux has Groups).
- For example an S3-admins group could be created for users who need to administer S3 buckets, but do not need access to any other part of the AWS platform,.

### IAM Roles

- An IAM *role* is a temporary identity that a user or service seeking access to your account resources can request.
- When creating a role, you begin by defining a *trusted entity* (four categories are listed below)
  - an AWS service
  - another AWS account (identified by its account ID)
  - a web identity who authenticates using a login (e.g. Amazon, Amazon Cognito, Facebook, or Google)
  - a Security Assertion Markup Language (SAML) 2.0 federation with a SAML provider.


- You might, for instance, need to give users logged in to your mobile application through their Google accounts access to specific resources on AWS services (such as data kept on S3).



## Authentication Tools

- AWS provides a wide range of tools to meet as many user and resource management needs as possible
- The Amazon Cognito, AWS Managed Microsoft AD, and AWS Single Sign-On services are for handling user authentication
- The AWS Key Management Service (KMS), AWS Secrets Manager, and AWS CloudHSM simplify the administration of encryption keys and authentication secrets

### Amazon Cognito

- Cognito provides mobile and web app developers with two important functions:
  -  you can add user sign-up and sign-in to your application (user pools)
  - you can give your application users temporary, controlled access to other services in your AWS account (identity pools)

- Defining a *user pool* requires you to set how you want users to identify themselves (such as an email address or username) as well as adding additional attributes such as address or birth of date. Password policies can also be configured and attached to the user pool.
- Setting up an identity pool requires defining from where the users can come from. e.g. using Cognito, AWS federated or even unauthenticated identities. An IAM role will be assigned to the pool, and once live, any user who identity matches the definitions will have the access to the resources specified within the role

### AWS Managed Microsoft AD

- Managed Microsoft AD is designed to have Active Directory control the way Microsoft SharePoint, .NET, and SQL Server–based workloads running in your VPC connect to your AWS resources
- It’s also possible to connect your AWS services to an on-premises Microsoft Active Directory using AD Connector

- Managed Microsoft AD is accessed through the AWS Directory Service
- Cloud Directory is a way to store and leverage hierarchical data like lists of an organisation's users or hardware assets

### AWS Single Sign-On

- AWS Single Sign-On (SSO) allows you to provide users with streamlined authentication and authorisation through an existing Microsoft Active Directory configured within AWS Directory Service.
- SSO also supports access to popular applications such as Salesforce, Box, and Office 365 in addition to custom apps that support SAML 2.0

### AWS Key Management Service (KMS)

- KMS deeply integrates with AWS services to create and manage your encryption keys, and provides a fully managed and centralised control over your system-wide encryption
- KMS ets you create, track, rotate, and delete the keys that you’ll use to protect your data
- For regulatory compliance purposes, KMS is integrated with AWS CloudTrail, which records all key-related events
- Key administration powers can be assigned to individual IAM users, groups, or roles

### AWS Secrets Manager 

- Passwords and third-party API keys for many of the resources your applications might need can be handled by the AWS Secrets Manager
- Secrets Manager you can deliver the most recent credentials to applications on request

- The Secrets Manager will even automatically take care of credential rotation

### AWS CloudHSM

- CloudHSM (where the HSM stands for "hardware security module") launches virtual compute device clusters to perform cryptographic operations on behalf of your web server infrastructure.
- CloudHSM provides the following functionalities:
  - Keys stored in dedicated, third-party validated HSMs under your exclusive control
  - Federal Information Processing Standards (FIPS) 140-2 compliance
  - Integration with applications using Public Key Cryptography Standards (PKCS) #11, Java JCE (Java Cryptography Extension), or Microsoft CNG (Cryptography API: Next Generation) interfaces
  - High-performance in-VPC cryptographic acceleration (bulk crypto)
- You activate an HSM cluster by running the CloudHSM client as a daemon on each of your application hosts.



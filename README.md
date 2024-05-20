- [IAM](#IAM)
- [EC2 Fundamentals](#EC2-Fundamentals)
- [EC2 Storage](#EC2-Storage)
- [EFS](#EFS)
- [AMI](#AMI)
- [ELB](#ELB)
- [Auto scaling group](#Auto-scaling-group)
- [RDS](#RDS)
- [Aurora](#Aurora)
- [Route 53](#Route-53)
- [VPC](#VPC)
- [S3](#S3)
- [CloudFront CND](#CloudFront-CND)
- [Docker](#Docker)
- [ECS](#ECS)
- [Beanstalk](#Beanstalk)
- [CloudFormation](#CloudFormation)
- [SQS](#SQS)
- [SNS](#SNS)
- [Kinesis](#Kinesis)
- [CloudWatch](#CloudWatch)
- [EventBridge](#EventBridge)
- [X-Ray](#X-Ray)
- [CloudTrail](#CloudTrail)
- [Lambda](#Lambda)
- [DynamoDB](#DynamoDB)
- [API Gateway](#API-Gateway)
- [CICD](#CICD)
- [SAM](#SAM)
- [CDK](#CDK)
- [Cognito](#Cognito)
- [Step functions](#Step-functions)
- [KMS](#KMS)

# uncategorized
Exponential backoff is used when you get a ThrottlingException, so on each retry double the seconds you wait
you must implement the retries on 5xx server errors
Dont implement retry on 4xx client errors

AWS Directory Services (AD)

Administrative overhead is the cost of setting up the service. This includes setting up users, security configs, and all the monitoring tools available. These questions usually differentiated themselves by having answers that would show you could setup a given task faster with a tool like cloud formation or beanstalk. They usually have an element of security to them as well.

Operational overhead is the cost of the day-to-day operation of the service in question. The questions usually differentiated themselves by having answers that demonstrated knowing a service could potentially be expensive and you might be able to minimize the cost using an a different AWS service or sometimes not an AWS service at all. These require general knowledge of all the different services available. Reading this subreddit it seems these questions take the full breadth of AWS services into account. Mine was heavy with Lambda.

Port 3306 is the default port used for the MySQL protocol

resource group:
If you work with multiple resources, you might find it useful to manage them as a group rather
than move from one AWS service to another for each task. maintain separate sets of resources for the alpha, beta, and release
stages. Each version runs on Amazon EC2 and uses an Elastic Load Balancer.
create a single page to view and manage all of the resources 

AWS Global Accelerator is a networking service that helps you improve the availability, performance, and security of your  
public applications. Global Accelerator provides two global static public IPs that act as a fixed entry point to your  
application endpoints, such as Application Load Balancers, Network Load Balancers, Amazon Elastic Compute Cloud (EC2)  
instances, and elastic IPs.   
good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP, as well as for HTTP use cases that  
specifically require static IP addresses or deterministic, fast regional failover.  

# IAM
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/3d22a197-c740-4bcb-bc14-38ea11d542e7">

### (identity and Access management)
It's a global service (doesn't matter in which region i create the user, available everywhere)

Composed of:
- Root account created by default (follow best practices)
- Users
  - can be in groups (can't contain other groups, not all users need a group, a user can be in multiple groups)
  - can be in Organizations

IAM Policy = permissions a user can be assigned a JSON document where we tell what they are allowed to do. least privilege best practice.
Consists of:
- "version": the policy version
- "Id": id for the policy (optional)
- "Statement": required
  - "Sid": identifier (optional)
  - "Effect": allow/deny access to certain APIs, ex: "ec2:StartInstances"
  - "Principal": account/user/role it's applied to
  - "Action": API calls that will be allowed / denied based on the effect
  - "Resource": list of res. to which the action is applied
  - "Condition": when it's applied (optional)
 
You can use Signature Version 4 to sign the API requests, to give API access in one AWS account to users in a different AWS account, also use  
resource policy on the resource you are trying to access...  

If there are 2 policies one the same resource Deny and one Allow, the Deny wins.
If IAM policy + S3 policy, the UNION of those creates the Total Policy Evaluated, so if you remove one the other one remains

Dynamic policies:
How do you assign each user a /home/user folder?
- either create an IAM policy allowing Matt access only to /home/matt ecc... policy for each user, but this doesn't scale
- or create a dynamic policy with IAM, using variables like ${aws:username}

Inline vs Managed policies:
- managed: maintained by AWS, good for power users and administrators, updated in case of new services and APIs
- Customer Managed Policy, granular control, versions, rollback, reusable, best practice
- inline: strict one-to-one if IAM deleted policy deleted

Permission to Pass a Role:
you need the iam:PassRole permission, roles can only be passed to what their trust allows  
The "trust policy" defines which resource can assume the role, and under which condition
The "Resource based" Policy can deny events from other AWS acc, or aggregate from multiple acc. into one using "Multi-Account aggregation"
The "access policy" when the user attempted to log in by using the provided credentials for the role

Defense mechanisms:
- password policy (length/special chars/numbers/pass expiration, prevent reuse)
- MFA

To access AWS:
- AWS management console (pw + MFA)
- CLI (access keys require an ACCESS KEY ID and a SECRET ACCESS KEY to make programmatic calls to AWS)
  - to use CLI with MFA you need to create a temporary session, you do so with the STS GetSessionToken API !! 
  - the CLI looks for credentials in this order: Command line, Env. Variable CLI credential file, CLI config file, Container cred. Instance profile cred.
- SDK (access keys require an ACCESS KEY ID and a SECRET ACCESS KEY to make programmatic calls to AWS)
  - looks for credentials in this order:
    - Environment variables
    - Java system properties
    - The default credential profile file.
    - ECS container credentials 
    - instance profile credentials (EC2 instances)  
      For ex: the AWS SDK for Java will find credentials stored in environment variables (where its allowed)
      before it checks for instance provide credentials (where its not allowd) and will allow access to
      extra S3 buckets.  

Access keys are generated with the AWS console, users manage their keys and are secret like passwords.  
connecting programmatically to AWS resources: Access key ID + Secret access key

IAM Roles:  
some AWS services will need to perform actions on your behalf, ex: you give permissions to your EC2 instance to do stuff on your AWS.  
to do so you need to assign a ROLE to EC2.  
If the application server is running on-premises you cannot assign an IAM role  

IAM security tools:  
- IAM Credentials report (list of account user status and credentials)  
- IAM Accesso advisor (shows the permissions granted to a user and when were last accessed)  

IAM Best practices: 
- Remove Account Access Key for the root account
- Use Temporary Security Credentials (IAM Roles) Instead of Long-Term Access Keys.
- don't use the root account except setup
- one phisical user per AWS user
- Assign users to groups and give permissions to groups, not each user
- Strong password policy
- MFA
- Don't embed access keys directly into code.
- Use different access keys for different applications.
- Use access keys for CLI and SDK
- Remove unused access keys.
- audit permissions w/ Credential report and IAM access advisor

Shared reponsability model:  
You know.

AWS STS  
Security token service: allows to grant limited and temp access to aws resources (up to 1hr)  
Define an IAM role, define its principals, use AWS STS to impersonate the IAM Role (AssumeRole API), Get the temp. credentials  
- AssumeRole: Assume roles within your account or cross-account, ex: use with cross-account to get temporary access to resources in a second account  
- GetSessionToken: for STS with MFA, from a user or AWS account root user. it returns Access ID, Secret Key, Session Token, Expiration date.
- GetCallerIdentityt: return details about the IAM user or role used in the API call
- DecodeAuthorizationMessage: decode error message when AWS API is denied.

# EC2 Fundamentals
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/70df717a-6b4b-4bcd-b73c-5371b49c5822">

### Elastic Compute Cloud
Instance types:
- General purpose (diverse workloads, websites or code repos)
- Compute-optimized (high level of processing, machine learning, gaming server ecc..)
- Memory-optimized (fast work for large data in memory, in memory dbs, cache store, real-time process of data)
- Accelerated Computing 
- Storage Optimized (when lots of datasets on local storage, data warehouse, online transaction)
- HPC optimized

m5.2xlarge: m = class like general purpose, 5 = generation (aws improve over hw over time) 2xlarge = size in instance class (the more cup and memory ecc..)

Security groups:  
firewall around EC2, control how traffic is allowed into EC2, they only contain ALLOW rules.   
they control:
- access to ports
- authorized IP ranges
- control of inbound network
- control outbound network

Ports to know
- 22 SSH secure shell (Linux instance access)
- 21 FTP file transfer protocol
- 80 HTTP unsecured websites
- 443 secured website
- 3389 RDP remote desktop protocol (windows instance access)

Connect with: SSH, putty, EC2 Instance connect

purchasing options:  
- on-demand
  - pay for what you use
  - no upfront commitment
  - recommended for short workload uninterrupted you don't know how the app. will behave
    
- reserved (long workloads 1-3 years)
  - discount compared to on-demand
  - you resource an instance attribute (type, region, os)
  - pay upfront, partial, no
  - recommended for steady-state usage (like a DB)
  - you can buy / sell it on the marketplace
    
- conv. reserved instances
  - change type but less discount
    
- savings plan (1-3 years you commit not to type but to amount of workload in dollars)
  - commit to a usage, beyond that its billed on-demand
    
- spot instances
  - most aggressive discount
  - you can lose it at any time, use it for workloads resilient to failure like batch, data anal, image process, distributed workload, flexible start and stop.
  - not suited for critical jobs
    
- dedicated instance
  - when compliance req. or bound sw licenses, or strong regulatory needs
  - on-demand or reserved
  - share host with others

- dedicated host
  - access to your own server and visibility on lower level

- capacity reservation
  - reserve on-demand instance capacity for any duration, you have access to ec2 capacity when you need it
  - no time commitment

EC2 user data:  
information that is parsed when the EC2 instances are launched. ex: run a shell script on Amazon EC2 instances each time they are launched

EC2 Instance metadata IMDS  
allows aws EC2 instances to learn about themselves without using an aws role for that  
HTTP..../latest/meta-data  
you can retrieve info about the EC2 but not the IAM policy ex: know the public IPv4 address

Use EC2 instance profiles is a container for an IAM role that you can use to pass role information to an EC2 instance when the instance  
starts. This is a secure way to authorize and EC2 instance to access AWS services.  

"--dry-run" argument Checks whether you have the required  permissions  for  the  action, without actually making the request  
 
# EC2 Storage
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/e04b234a-8821-4d49-94e8-9295df6a2c08">

EBS (elastic block store) network drive you can attach to your drive while they run: / block-level storage service
  - persistent data, only mounted to 1 ec2 at the time
  - locked to only 1 AZ at the time
  - network drive so a bit latency to communicate (not physical drive)
  - because its network it can be detached and reattached somewhere else
  - can have a delete on termination attribute

EBS Volume types:
  - gp2/gp3 "general purp." balance price and perf for wide variety workloads
    - can be used for boot volume
    - cost-effective low latency
    - virtual desktops and test environments
    - gp3 can individually increase IOPS and throughput, for gp2 they are linked together
      
  - io 1 / io 2 "prevision iops": highest performance SSD
    - can be used as boot volume
    - for mission-critical low latency or high throughput workloads
    - good for DB workloads (sensitive to storage and performance)
      Multi attach family
      - attach to multiple EC2 in same AZ (only available to io 1 / io 2)
      - for high app. availability
      - manage concurrent write operations
      - MAX 16 EC2 at the time
        
  - st 1 low-cost hdd for frequent access throughput intensive
    - cant be a boot volume
    - throughput optimized (big data, data warehouse)
      
  - sc 1 lowest cost hdd for less frequent access
    - cant be a boot volume
    - for data that is infrequently accessed
    - when the lowest cost is important

EBS snapshot:
backup of EBS at a point in time:
  - can copy snapshot to another AZ, thats how you transfer it to another AZ
  - can move it to an "archive tier" that is cheaper, 24-72hrs to restore it
  - recycle bin, can be recovered
  - fast snapshot restore

EC2 instance store:
  - better I/O performance, high performance because physically attached hw store
  - lose the storage if ec2 is stopped
  - good for buffer / cache / scratch data / temp content
  - risk of data if data center hw fails
  - your responsibility to backup

# EFS
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/02c35483-16c1-4347-833b-cafe9c6dc073">

Elastic file system (network file system can be mounted on many EC2) / file storage service (not object storage service
or block-level storage service)
  - they work with EC2 in multiple AZs
  - high availability, pay-per-use
  - security group to control access to EFS
  - compatible with AMI
  - encryption at rest with KMS
  - Scale automatically and is pay-for-use
  EFS Performance classes:
    - EFS scale
    - performance mode
    - throughput mode
  EFS Storage classes:
    - standard: frequent access
    - EFS-IA infrequent access, lifecycle policy says after how much it gets converted
    - EFS One Zone-IA best cost-saving
    

# AMI
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c8c0f021-4aef-4778-827b-697415ef7520">

### Amazon machine image
it powers the EC2 instance, represents a customization of EC2 instance
you can launch EC2 from
  - a Public AMI: aws provided
  - your own AMI
  - AWS marketplace AMI

how to create:
  - start EC2 instance and customize it
  - stop it
  - build the AMI
  - launch instance from other AMI

# ELB
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/6e55cd0d-248e-4592-9a70-494a59b41e7d">

### Elastic load balancer
it's a MANAGED load balancer, AWS guarantees it will be working and will upgrade it automatically  
what it does:  
- forwards traffic to multiple servers
- expose a single point of access (DNS) to your app
- handle failures of instances, using health checks, by using a port and route to check the HTTP response.
  ex response 400 not healthy


4 types:
- classic load balancer: deprecated

- application load balancer (ALB): routs https, websocket traffic
  -  load bal. to multiple http apps. across machines (target group), ec2 instances/ec2 tasks/lambda
  -  load bal. to multiple apps on the same machine (containers)
  -  supports route routing / query strings like example.com/users
  -  we can do stuff like if the request is ?=mobile it redirects to a specific instance otherwise if ?=desktop to
     different one
  - You can register your Lambda functions as targets and configure a listener rule to forward requests to the target group for your
    Lambda function. When the load balancer forwards the request to a target group with a Lambda function as a target, it invokes  
    your Lambda function and passes the content of the request to the Lambda function, in JSON format.  
    X-Forwarded-Proto: protocol (HTTP/HTTPS)  
    X-Forwarded-Host: original Host header requested by the client  
    X-Forwarded-For: original IP address of a client  
    X-Forwarded-Port header: original port that the client used to connect  

- network load balancer (NLB): forwards TCP UDP LTS traffic
  - works like app. load balancer, we redirect to certain target groups
  - redirects to target EC2 instances
  - redirects to IP addresses
  - supports TCP,HTTP,HTTPS health checks
  - capture the user's source IP address and source port 
    
- gateway load balancer (GLB): operates at layer 3 network layer, scale and manage 3rd party virtual applications
  for example firewalls, intrusion detection system, inspection system... for example if u want your traffic to be
  inspected before it reaches EC2
  - GENEVE protocol 6081
  - target adresses: EC2 / IP adresses

Sticky sessions: its possible to implement so that the same client is always redirected to the same instance behind a  
load balancer, it uses a cookie with an expiration date. it can be an Application cookie or duration-based cookie.

Cross zone load balancing: each load balancer distributes evenly across all instances in all AZs  
or without all ASz receive the same amount no matter how many instances in the AZ.

can be public or private

SSL/TLS certificate allows traffic between your client and load balancer to be encrypted in transit, they are
issued by Certificate Authorities, they have an expiration date

Connection Draining: If EC2 is shutting down it goes in draining mode so the uses that are already connected and need  
to complete their requests finish and ELB doesn't forward new traffic there, then when it's all done it shuts down.

load balancer security group allows traffic from anywhere, in routing table 0.0.0.0/0
EC2 security group allows traffic only from the load balancer, so in routing table the source is the load balancer name

# Auto scaling group
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/f861c0d3-3b2a-4546-a6fd-2ab0a6252823">

in cloud you can get rid or add servers quickly  
the goal of ASG is to   
- scale out (add EC2) to match increase in load
- scale in (rem. EC2) to match decrease in load
- ensure we have a min and max number of EC2 running
- registered new instances to ELB
- ASG launch template contains info on how the new instance should be created
- it's possible to scale based on CloudWatch alarms metrics
- after a scaling happens there is a default 300 sec cooldown period

Scaling policies:  
- dynamic scaling: based on tracking scaling ex: target to have ASG CPU to stay at 40%
- simple / step scaling: when a CloudWatch alarm is triggered
- scheduled scaling: anticipate scaling based on pattern
- predictive scaling: pattern that repeat based on forecast

An Auto Scaling group has a maximum capacity of 3, a current capacity of 2, and a scaling policy that adds 3 instances  
Amazon EC2 Auto Scaling adds only 1 instance to the group.  

Instance refresh:  
if you want to update the launch template, as scaling happens old instances get terminated and new ones  
will have the new template, so at the end after a while all instances will have the new template

# RDS
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/b70b6dcf-bc97-462c-8db3-30509fcb3ed4">

### Relational database service
Managed DB service, uses SQL, because its managed, rather than EC2 own DB:  
- OS patching by AWS
- continuous backups and restore
- read replicas: help you scale your reads (not writes)
  - async replications so the reads are "eventually consistent" because if they haven't finished syncing you can read
    old data sometimes
  - up to 15 read replicas
  - cross AZ or cross-region
  - can be converted to their own DB
- multi-AZ setup
  - failover if disaster recovery
  - replicas can be used for multi-AZ disaster recovery (THEY ARE NOT READ REPLICAS)
  - one DNS name, automatic failover
  - sync replication
  - Amazon RDS applies operating system updates by following these steps:
    Perform maintenance on the standby.
    Promote the standby to primary.
    Perform maintenance on the old primary, which becomes the new standby.
- can enable AUTO SCALING!
- storage backed by EBS

RDS proxy:  
fully managed db proxy for RDS  
- Allows apps to pool and share DB connection, minimizing timeouts, open connections, and reduce stress on DB resources
- Auto scaling, serverless, across multiple AZ
- Apps connect to proxy not directly to DB instance

# Aurora
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c84aa5c9-1953-48d1-8304-e702b1b3c738">

proprietary tech from AWS  
- SQL DB, 5x & 3x times faster
- The storage automatically grows
- up to 15 replicas
- automatic native failover
- automatic backup and recovery
- 6 copies of data in 3 AZ
- 1 of the 6 replicas in a master that writes
- 5 of the 6 are read, you use a Read Endpoint that connects balances the request and decides from which one to read
- automatic patching /maintenance

both RDS and Aurora:  
- at-rest encryption: with AWS KMS
- at-flight encryption using AWSTLS
- IAM authentication: IAM roles to connect / pw username
- security groups for network access
- audit logs can be enabled

Elastic cache:  
Redis / memcached  
in-memory database with high-performance low latency  
- is an in-memory KEY-VALUE store that has no way to deliver static HTTP content
- common query will be cached
- it's managed, so aws takes care of the usual stuff
- Application queries ElastiCache, if available gets from there, otherwise query RDS and store in ElastiCache
(cache hit or cache hit)
- or you can cache the session data for faster
- cached data may be eventually consistent
- efficient if data changes slowly
- Lazy loading / Cache-Aside / Lazy Population architecture: cache hit / cache miss depending if the data is in ElastiCache, if not get from RDS and write it to cache for further searches.
  When your application needs to read data from the database, it checks the cache first to determine whether the data is available.
  Loads the data into the cache only when necessary (if a cache miss occurs).
  Lazy loading avoids filling up the cache with data that won’t be requested.
  cache can become stale if Lazy Loading is implemented without other strategies (such as TTL).
- write through architecture: write to cache when there is a modification to the DB

Redis: Supports more complex data structures, You can use Amazon Elasticache for Redis Sorted Sets to implement a dashboard with the ability to sort or rank the cached datasets.  
session state data be maintained externally, whilst keeping latency at the LOWEST possible value (like DSD): The two options presented in the answers are Amazon  
DynamoDB and Amazon ElastiCache Redis. ElastiCache will provide the lowest latency as it is an in-memory database.  
Memcached: Primarily supports string-based keys and values; does not support advanced data structures. Supports multi-threading, NOT as highly available as Redis   

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/84dafa7c-cfac-4a5c-934e-636a2badf4ed">


Cache eviction:  
- you delete item explicitly
- evicted cuz memory is full and not recently used
TTL time to live:
- used for leaderboards / comments / activity streams
- can range from a few seconds to days

# Route 53
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/72e0e111-ab7b-44fc-a095-c86f2704330b">

DNS domain name system: translates ip addresses to human-friendly names  
terminology:
- Domain registrar: Route 53, godaddy..
- DNS records:
- zone file: contains DNS records
- name server: resolver DNS queries
- top-level TLD, second-level SLD  
<img width="450" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/d48cb9cf-8a74-4eb9-a5ba-182dd28d3a23">

Route 53 is an authoritative DNS (you can update the records yourself), and is a Domain registrar  
Route 53 records contains:  
- Domain/subdomain name: example.com
- record type: (must know)
  - A maps hostname to ipv4
  - AAAA maps hostname to ipv6
  - CNAME Point a hostname to another hostname, works if you have a non-root domain (something.domain.com). provides a domain name for it, such as d111111abcdef8.cloudfront.net. Instead of using this provided domain name, you can use an alternate domain name (also known as a CNAME).
 
To ensure ongoing connectivity the Developer needs to use an Elastic IP address for the EC2 instance and DNS A record as this is  
the only type of static, public IP address you can assign to an Amazon EC2 instance.  
- Public – public address that is assigned automatically to instances in public subnets and reassigned if instance is stopped/started.  
- Private – private address assigned automatically to all instances.  
- Elastic IP – public address that is static.  

To use your own domain name, such as www.example.com  
  - Alias: Point a hostname to AWS resource, works with root name (ELB, CloudFront, API Gateway, s3, VPC)
  - NS name server for the hosted zone controls how traffic is routed
    -  public hosted zones responds with the public ip when trying to resolve the DNS
    -  private hosted only you in your VPC network can access it
- value: 128.241.223.111
- routing policy: how r53 responds to queries
  - Simple, route traffic to a single resource, if multiple values (IP addresses) are returned the client chooses a random one
  - Weighted: routing policy control the % of the requests that go to each resource
  - Failover: done with health checks, if one is not healthy, switch
  - Latency-based: redirect to the resource that has the least latency
  - Geolocation: based on where the user is located
  - Multi-Value: used when routing to multiple resources
  - ip-based: routing is based on clients IP address
  - Geoproximity: based on a value called bias, which is the radius  
- TTL amount of time the record is cached at DNS resolver

traffic flow: node-based editor allows to config complex routing decision policy 

Health checks are for public resources
- this allows automated DNS failover

Route 53 Records TTL:
- the client will respond with the cached IP if TTL hasn't expired yet.

domain registrar: allows you to buy your domain value (GoDaddy) and they give you usually a way to manage your domain record  
but you can use a different one if needed

# VPC
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/99820e8a-bbf2-41f7-aca8-44d30ce6a14e">


### virtual private network
subnet allows to partition network inside VPC  
subnet can be public / private  
to define access between subnets and how it flows between sections use Routing Tables  

Internet gateway: help VPC instances connect to internet

Public subnet has a route to the internet gateway

NAT gateway (aws managed) and NAT instance (self-managed) allow your instances in private subnet to access the internet while remaining private

network ACL (NACL) firewall which controls traffic from and to subnet  
can only allow or deny rules  
is stateless return traffic must be explicitly allowed  

security group: only the allowed rule  
firewall the controlled traffic from EC2 instances   
stateful return traffic is automatically allowed  
timeout errors if no access allowed  

VPC flow logs:  
capture info about IP traffic from network interfaces, log data can be published to Amazon CloudWatch Logs or Amazon S3

VPC peering: connect 2 VPCs privately  
must not have overlapping CIDR

VPC endpoints:  
allow to connect to AWS services using a private network instead of public network

connect on-premise site with VPC:  
VPN encrypted connection over public connection, tunnel to VPC from a location OUTSIDE of AWS, you cant use them in a subnet for example  
Direct connection: physical connection to AWS

3 tier solution architecture:  
Route 53 -> ELB public subnet (1) -> private subnet (2) with 3 EC2 -> data subnet (3) RDS / elastiCache

LAMP stack on EC2  
Linux  
Apache  
MySQL  
Php  

WordPress on AWS  

# S3
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c0ddee13-1fc0-4509-a333-766fd99bd775">

Used for everything, is the backbone for a lot of internet stuff  
backup and storage, disaster recovery, archive, hybrid cloud storage, app hosting, media hosting, data lakes and big data, sw delivery, static website  

store objects (files) in "buckets" / object storage service
must have unique name  
defined at the region level ( not global level)  

contains:
- each object has a key: the full path of the file, prefix + object name
- max size is 5TB, files > 5GB are multi part upload
- metadata list of key-value per object
  - name must begin with "x-amz-meta-"
  - retrieved when retrieving the object
  - cant filter by metadata
- tags key-val for security / lifecycle
  - useful for fine-grained permission
  - cant filter by tag
  - good for analytical purposes
- version ID (if versioning enabled) 

security:
- IAM policy for users
- Bucket policy; bucket-wide rules at the resource level
- object ACL
- bucket ACS
- encrypt the object server side SSE 
  - SSE-S3 server-side encryption with s3-managed-keys
    - key handled and managed by AWS
    - AES-256
    - enabled by default
  - SSE-KMS server-side encryption with KMS keys stored in aws kms
    - user control over keys, can audit who uses it with Cloudtrail
  - SSE-C server-side encryption with customer-provided keys
    - AWS does not store the key you provide
    - must use HTTPS
    - The GetObject API call must specify the customer-provided encryption key that is used to initially encrypt the data  
      and the Object key
- encrypt the objects client-side
  - encryption and decryption outside of AWS
- encryption in transit
  - SSL/TLS encryption while its being transmitted, using the HTTPS endpoint
    
can force encryption with bucket policy  

x-amz-server-side-encryption header to request server-side encryption with S3 Managed keys.  
x-amz-server-side​-encryption​-customer-algorithm, x-amz-server-side-encryption-customer-key and x-amz-server-side-encryption-customer-key-MD5 headers. Headers for Customer managed keys.  

SecureTransport:  
policy permission to limit access by IP addresses or require that data must be encrypted in transit (HTTPS).

- CORS: cross-origin resource sharing
  it's a security in web browsers to allow requests from other website origins
  (popular question) if a client makes a cross-origin request on our S3 bucket we need to enable the correct CORS headers
  "Cross Origin Resource Sharing" must be enabled on the resources that shares
  
- MFA delete
  require MFA when deleting an object

- S3 access logs
  you can enable logging of all accesses and operations. The target of logs should be a different S3 bucket never the same one

- pre signes URLS give access to a file on a private bucket from 1min to 12hrs, using a pre-signed url by you

- S3 access points:
  An S3 Object Lambda access point is a new type of access point that you can create to invoke your own AWS Lambda function to modify the content of an S3 object.
  You can use S3 Object Lambda access points to transform data as it is being retrieved from an S3 bucket, without modifying the original data stored in the bucket.
  - Or Access point policy: Users from finance can only access /finance/... files, sales can only access /sales/... files
  this is done with an access point policy.

- S3 Object lambda
  if u want to run a function before its being retrieved, use and object lambda access point.
  ex: adding watermark specific for the user, resizing, converting the data

S3 can host static websites and have the accessible public with a policy  
Enable public access and grant everyone the s3:GetObject permissions  
Upload an index document and enter the name of the index document when enabling static website hosting  
The error file is optional

you can version your files, it's enabled at the bucket level, instead of overwriting it creates a new version  
you can enable s3 versioning to have objects that when are deleted are hidden and can be recovered

S3 can generate pre-signed URLs to issue a request as the person who signed it, using the IAM key of the signing principal, it has a limited lifetime

replication:
enable versioning 
- CRR cross-region replication (compliance or low latency)
- SRR same region replication (live replication, log aggregation)
- buckets can be in different AWS accounts
- copy is async

Storage classes:
- general purpose
  - frequently accessed data
  - low latency high throughput
  - for big data analytics, mobile gaming, content distribution
- Standard-IA
  - For data that is less frequently accessed, but requires rapid access when needed
  - for disaster recovery / backups
- One Zone-IA
  - in a single AZ
  - secondary copy of backup or data u can recreate
- Glacier Instant Retrieval
  - millisecond retrieval
- Glacier flexible retrieval
  - 1-5 min
- glacier deep archive
  - 12 hrs
- intelligent tiering
  - move objects between tiers based on usage
 
To avoid throttling in Amazon S3 you must ensure you do not exceed certain limits on a per-prefix basis. You can send 3,500  
PUT/COPY/POST/DELETE or 5,500 GET/HEAD (read) requests per second per prefix in an Amazon S3 bucket. There are no limits to the  
number of prefixes that you can have in your bucket

durability: how many times an object is going to be lost 9.99 11 9s  
availability: how readily a service is: 99.99% not available for 53min

can use lifecycle configurations or move between classes manually  
lifecycles rules:  
- Transition Actions: configure objects to transition to another storage class after a certain time
- Expiration actions: configure objects to expire (delete) after a certain time / delete old files / del partial files

S3 Event notifications:  
like generating thumbnails of images uploaded to s3  
S3 bucket -> Amazon Event Bridge -> rules -> services  
S3 event notifications can directly invoke a Lambda function, you dont necessarely need Event Bridge  

S3 performance:  
- can speed up upload by using multi-part upload
- S3 transfer acceleration: fast, easy, and secure transfers of files over long distances between your client and and S3  
  name of the bucket used for Transfer Acceleration must be DNS-compliant and must not contain periods (".").
- byte-range fetch: request a specific range of bytes in a file to speed up downloads (only partial data)
- SQL server-side filtering, the server sends data already filtered

- S3 static websites use the HTTP protocol only and you cannot enable HTTPS. To enable HTTPS connections to your S3 static website,  
  use an Amazon CloudFront distribution that is configured with an SSL/TLS certificate. This will ensure encrypted in-transit as per the requirements

syntax to represents to Object
- s3:// dctlabs/Development/Projects.xls this is the full path to a file including the bucket name and object key
- Development/Projects.xls subfolders/direct name rapresents an Object key 
- "Project=Blue" is an example of an object tag. You can use object tagging to categorize storage. Each tag is a key-value pair.
- "arn:aws:s3:::dctlabs" is the Amazon Resource Name (ARN) of a bucket.
  (ARNs) uniquely identify AWS resources. We require an ARN when you need to specify a resource unambiguously across all of AWS,
  such as in IAM policies, Amazon Relational Database Service (Amazon RDS) tags, and API calls.
  arn:partition:service:region:account-id:resource-id
  arn:partition:service:region:account-id:resource-type/resource-id
  arn:partition:service:region:account-id:resource-type:resource-id

# CloudFront CND
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/eee7115d-40b2-432c-8b06-0d267cd3e9cc">

It's a CDN content delivery network  
improves read performance, content is cached at the edge  
DDoS protection integrates with shield  
It integrates with ANY HTTP backend you want, S3, app load balancer, EC2, S3 website  
Client asks to Edge location (cache), if it doesn't have it the edge location gets it from the origin  

If you want In-flight encryption select these settings of Origin Protocol and Viewer Protocol.  
IN EXAM IF CloudFront with SSL / Secure connection use this!!!

<img width="241" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/2ddcc116-6b9b-49b0-9bc9-8f7fd65e4ce7">

<img width="307" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/62c2db31-7806-4536-a120-c2662186b8e0">


CloudFront vs something like S3 replication  
CloudFront:  
- is global edge network
- files are cached to each edge location TTL for a day
- great for static content that must be available eveywhere
S3 replication
- must be setup for each region you want replication
- files are updated in near real-time
- great for dynamic content that need to be available at low latency in few regions

The cache lives at each CloudFront edge location  
you want to maximize the cache hit ratio  
objects are identified using the Cache Key  
- unique ID for every object
- consist of hostname + resource portion of URL
- if you want info not static, like based on user device / language /location add other elements to HTTP headers using CLOUDFRONT CACHE POLICIES
  - define cache based on HTTP headers, cookies, query strings
  - controls the TTL
  - All info you include here is automatically included in the origin requests
  - YOu can use an Origin Request policy for values that you want to include in origin request without including them in cache key  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/26b2a180-f877-462e-bba7-f443c36d60cc">

Cache invalidation:  
If you update your backend origin, CloudFront doesn't know about it and will only change after the TTL  
You can force and entire or partial cache refresh bypassing the TTL using cloudfront invalidation  

you can config different cache settings for different content types or path patterns (ex maximize hits by separating dynamic and static content)  

ALB or EC2 as HTTP background  
instances must be public there is no private VPC connection from edge location  
so the resources must allow traffic from public IP of edge location  
If it's an ELB it must be public, if its EC2 it can be private because there is VPC connection from ELB to EC2  

Geo restriction  
allowlist: if you are on the list of approved countries  
blocklist prevents user fron entering from banned countries  

Signed URL / signed cookie  
Signed URL = access to individual files ( one URL per file ) To restrict access to content 
Signed Cookies = access to multiple files at once  
for example premium private users who paid for shared content  
we can use a URL / Cookie we attach to the policy that includes:  
- url expiration
- ip range
- who is the trusted signer

- For S3 you can create a special CloudFront user called an origin access identity (OAI) and associate it with your distribution. Then you configure permissions
  so that CloudFront can use the OAI to access and serve files to your users.

CloudFront signed URL vs S3 pre-signed URL  
pre-signed you have access to directly the S3 without being able to use Cloudfront, it issues a request simulating the person who pre-signed the URL

Signed URL process:  
2 types of signers, either a trusted key group ( recommended ) or an AWS account that contains a key pair (not recommended)  
- create one of more trusted key groups, the public key is used in the URL, the private key is used by your app.
- When you create a signer, the public key is with CloudFront and private key is used to sign a portion of URL  
- When you use the root user to manage CloudFront key pairs, you can only have up to two active CloudFront key  
  pairs per AWS account  


Pricing:  
the cost of data out per edge location varies  
you can reduce the number of edge locations for cost reduction using the 3 Cloudfront price classes

multiple origins:  
CloudFront with multiple origins to serve both static and dynamic content at low latency to global users  
can help access the static and dynamic content while keeping the data latency low  
route to different kinds of origins based on the content type or path  
for example /API/* requests from ALB, /* requests from S3 bucket

origin groups:  
to increase high availability and do failover  
one primary and one secondary origin, if the primary fails the secondary is used

Filed level encryption:  
protect user-sensitive info through application stack  
sensitive info encrypted at the edge close to user

Real-time logs:  
get requests recieved by CloudFront and sent to Kinesis data streams, here you can monitor, analyze and take action based on content performance

# Docker
- software development platform to deploy applications
- apps are packaged in containers that can be run by any OS, they run the same regardless of where they run
Docker VS VMs  
resources are shared with the host, many containers on one server, they can be managed by ECS, EKS, ECR, Fargate  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/25b17b94-3cab-4d38-b615-95f0e1fe9864">

Elastic Beanstalk uses the "docker-compose.yml" file to pull and run your image if you are using Docker Compose.  
Otherwise Docker platforms for Elastic Beanstalk You specify images by name in the "Dockerrun.aws.json" file and save   
it in the root of your source directory.

Docker builds images automatically by reading the instructions from "Dockerfile" which is a text file named Dockerfile  
that contains all commands, in order, needed to build a given image.

# ECS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/7701684b-84d8-4979-bf61-f32db171ac1a">

### Elastic container service

ECS agent facilitates the communication between your container instances and Amazon ECS  
/etc/ecs/ecs.config contains cluster Parameter values, like cluster name.  

EC2 Launch type:  
to launch a container = to launch an ECS task  
If you decide to use an "EC2 launch type" (EC2 inside ECS and ECS agent with docker on each EC2) You must provision and maintain the EC2 instances that run inside ECS
- the EC2 instance makes API calls to ECS services
- conds container logs to CloudWatch
- pulls Docker images from ECR
- individual tasks can have roles to API calls with AWS services
- Load balancing: Dynamic host port mapping, each task has a different random port,
  the ELB finds the correct ports automatically even if random. You must allow on the EC2 instance security group any port from the ALB security group
- for EC2 instance storage to share data between tasks
- ECS Task Placement Strategy:
  when a task of EC2 is launched, ECS must determinate where in which EC2 to place it for memory and CPU availability
  to assist you, you can define a Task placement strategy and task placement constraints
  - Binpack: Place tasks based on the LEAST available amount of CPU or memory, cost saving so it uses the least possible amount of EC2s
  - Random: places a task randomly, no logic to it
  - Spread: Places tasks evenly based on a specified value: ex spread evenly along AZs
- ECS task placement constraints:
  - distinct instance: place each task on a different container instance, so never 2 tasks on the same container
  - memberOf: place a task on the instance that satisfies an expression

Another type of "launch type is *Fargate*", you don't need to provision the infrastructure, serverless, 
just run EC2 tasks, automatic scaling, is fully managed
- load balancing: each task has a unique private ip, you only define the container port. the ALB will
  connect all incoming connections to the same port.
- Fargate ephemeral storage tied to lifecycle, shared between containers
- ECS task placement is automatically handled
- Charge you for running tasks rather than running container instances
  
Load balancer integration:  
we can run an ALB for different instances with tasks

Cluster same port:  
set the host port number to 0 and ECS will automatically assign an available
port. We also need to assign port 80 to the container port so that the web service is able to run.

Data volumes:  
we can use EFS file system onto ECS to have a file system for all tasks, tasks running in any AZ will share the same data in the EFS. 
Fargate + EFS = serverless, you cant mount s3 as a file system here

ECS service auto scaling:  
automatically increase / decrease tasks  
we can scale on CPU / Memory / ALB requests  
ECS services to scale underlying EC2 instances:  
- ECS Cluster Capacity Provider is what is used to automatically provision and scale the infrastructure,
  adds EC2 when you are missing capacity (CPU, RAM) (recommended) this is used to scale the cluster
  container instances, not the number of tasks. To scale nr of tasks Create an ECS Service with Auto Scaling and attach an Elastic Load Balancer
- Auto scaling group scaling, scale based on CPU utilization
we can scale using  
- target tracking based on a target value from Cloudwatch metric
- step scaling scale based on a specified Cloudwatch alarm
- scheduled scaling scale based on a specified date / time

ECS Rolling updates:  
when upgrading from V1 to V2 we can control how many tasks can be started and stopped and in which order in order  
to roll updates to different tasks by starting the new ones with v2

ECS solution architectures:  
serverless object processing example with ECS  
user uploads image to s3 bucket  
even bridge runs ECS2 task  
the task now gets the object from the bucket  
it saves the results in Dynamo DB  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/46bfc7f7-1b41-4450-818b-6a312fb2323a">

another serverless example using scheduled task every 1 hr  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/0ec7cc33-ad03-47ce-92d6-0da32133f6c8">

tasks pool from SQS and if more messages, use auto-scaling to handle them  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/df9fc722-bac7-4d56-b049-bdfec728587e">

Event bridge allows you to handle lifecycle of your tasks  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/d2c60519-8734-4fb9-af5f-87043fc47de8">

ECS task definition:  
metadata JSON tells how to run Docker containers like AMI for ECS  
- port binding
- CPU / memory
- environment variables: hard coded like URLs, SSM parameters (eg API keys, shared configs). You fetch either from SSM parameter store or Secrets manager
- IAM roles: task definition, each tasks in a service gets this role, you can define different roles per service
- logging config
- network info
- Data volumes: share data between containers, Specify both containers in the definition. Mount a shared volume, so like some task can write to a path, and a logging task can read from it and save them.
  for EC2 instance storage, for Fargate ephemeral storage tied to lifecycle. Create one task definition.

ECS Cluster queries are expressions that enable you to group objects. For example, you can control the placement of tasks onto groups container instances by attributes  
such as Availability Zone, instance type, or custom metadata. 

Amazon ECR:  
elastic container registry, used to store and manage docker images on AWS (IT CAN HOST THEM)    
private repo for your account or public repo using ECR public gallery  
IAM role to EC2 instance to pull ECR repository  

AWS Copilot:  
CLI tool to build release and operate containerized apps.  
it provisions all required infrastructure for containerized apps (ECS, VPC, ELB, ECR...). Auto-deploy with code pipeline + deploy to multiple environments

Amazon EKS:  
another way to run containerized apps. using Kubernetes not ECS, runs docker containers but is open source and used by more different cloud providers  
same goal but different API. supports EC2 mode or FARGATE mode.   
It's Cloud agnostic so it can be used in any cloud, use it if your company / another cloud you use, uses it.  
You create EKS nodes with tasks inside, they can be: Managed node groups (auto-scaling, on-demand ec..) by AWS or Self-Managed Nodes.  
EKS data volume supports EBS, EFS, FSx, by specifying a StorageClass. It uses a Container Storage Interface CSI, to be able to use them  

You terminated the container instance while it was in STOPPED state, that lead to this synchronization issues,  
container instance isn't automatically removed from the cluster.  
If you terminate a container instance in the RUNNING state, that container instance is automatically removed,  
or deregistered, from the cluster.   

Container Agent:  
allows container instances or EC2 to connect to your cluster.  

# Beanstalk
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/2902359b-4e68-430a-b99a-a9da6e41c116">

Allows you to skip the manual setup for each service when deploying an architecture.  
It's a managed service that uses components seen before (EC2, ASG, ELB, RDS...) but you don't manually connect them and configure them, you just focus on the code development  
we still have full control over the config if needed. The service is free but you pay for the underlying resources it deploys.  
Components:  
- Application: is the collection of all components
- App. Version: version of your app. code
- Environment:
  - contains the collection of AWS resources running
  - tiers: services tiers
  - you can create multiple environments

Example of architectures:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4dc3c7dd-353e-4781-8466-769581ccbef4">

deployment mode:  
single instance: great for dev  
High availability with load balancer: great for production  
AWS Elastic Beanstalk makes it easy to create new environments for your application. You can create and manage  
separate environments for development, testing, and production use, and you can deploy any version of your application  
to any environment  

You can add AWS Elastic Beanstalk configuration files (.ebextensions FOLDER) to your web application's source code to configure your environment and customize   
the AWS resources that it contains. YAML or JSON-formatted documents with a .CONFIG file extension that you place in a folder named .ebextensions and deploy (EXCEPT FOR ALREADY-EXISTING "env.yaml" config file for the environment.)  

Deploy options for updates:  
- All at once, deploy the new v all in one go but it has a bit of downtime  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/28d406a4-c6d7-4b02-84cb-74e1349db53b">
  
- Rolling: update a few instances at a time (at one point it runs at a smaller capacity)  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a16b4ed3-21aa-48a3-b73b-96fe01d23ddb">
  
- Rolling with additional batches: like rolling but actively starts new instances to switch, so it has all the capacity when needed  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/2cb0caf7-d0db-43b9-ad76-f62eeb576408">
  
- immutable: deploys all to new instances the switches everything at once when it's ready (QUICKEST ROLLBACK IF FAILURE, but longest deploymeny, zero downtime, CHEAPER THAN ROLLING WITH ADDITIONAL BATCHES)  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/cc51490f-db72-457f-8968-4b756dcd4f67">
  
- blue green: create a new environment and test, and switch when ready  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/051954d9-e3eb-4e27-8f85-60f0c180447b">
  
- traffic splitting: sends a small % of traffic to new deployment  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/07331a37-8b38-4f7f-850e-97060e09a667">

Elastic Beanstalk CLI, helpful in automating pipelines  

EB Lifecycle policy:  
policy to remove old versions  

EB extensions:  
a file in the .ebextentions/ directory in our code that can configure all the parameters you find in the web UI  
Because it uses CloudFormation UNDER THE HOOD, the extensions provision any service you want.  

EB cloning:  
clones an environment. with the exact config. useful for "test" versions of app. After cloning u can change settings.  

EB migration:   
for example ELB can't be changed (only configured) after cloning, so we need to migrate:  
create a new env. with the same config except LB (cant clone)  
deploy the app onto new env. Perform a swap.  

Your source bundle must meet the following requirements:  
- Consist of a SINGLE ZIP file or WAR file
- Not exceed 512 MB
- Not include a parent folder or top-level directory (subdirectories are fine)

worker environment:  
If your application performs operations or workflows that take a long time to complete, you can offload those tasks to a dedicated worker environment.  
Decoupling your web application front end from a process that performs blocking operations is a common way to ensure that your application stays responsive under load.

# CloudFormation
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/1fe959c9-96f0-4dd9-a07c-82fcd3c052e7">

Declarative way to outline your AWS infrastructure for any resource  
infrastructure as code  
can create a template in a Manual way: edit them in Cloud Formation Designer or code editor  
automated way: using AWS CLI  

YAMS and JSON are languages you can write CF templates  
"aws cloudformation deploy" then "sam deploy" commands  

template components:
- template version
- description
- resources: are the core of your CloudFormation template, represent the components that will be created, res. can refer each other
- parameters: they are a way to provide inputs to your AWS CF template (ex choose type of ec2, password...)
- mapping: fixed vars within CF template ex (dev vs prod) The optional Mappings section matches a key to a corresponding set of named values. For example, if you want to set values
  based on a region, you can create a mapping that uses the region name as a key and contains the values you want to specify for
  each specific region
- outputs: output the vars such as VPC ID and subnet ID to network stack
- conditions: control the creation of res. based on conditions ex: what region you deploy in
  (Parameters section can't be associated with Conditions)
- Instrinct functions (the most important):
  - Ref: returns the instance ID
  - GetAtt: returns the value of a specific attribute, ex: when creating and EBS vol you want to get the AZ of the EC2
  - FindInMap: returns a named value from a specific key
  - importValue: import values that are exported in other stacks
  - Condition Functions: if / and / not / or
  - Base64: convert value to base64

CF Rollbacks:  
if a stack creation fails you have the option to default: everything rolls back, or to disable rollback to troubleshoot

CF Change Set:  
summary of proposed changes to an AWS CloudFormation stack without implementing the changes in production 

CF Service Role:  
IAM role that allows CloudFormation to CRUD resources, gives users the ability to CRUD even if they don't have permission to work with the resources in the stack

CF Capabilities:  
CAPABILITY_IAM to enable CF to create or update IAMs  
CAPABILITY_AUTO_EXPAND enable if you need stacks within stacks for dynamic transforms of the CF  
insufficientCapabilitesException: exp. thrown if the capabilities haven't been acknowledged when deploying a template.

CF Deletion policy:  
Control what happens when template is deleted or when a res. is removed from CloudFormation template  
- Default DeletePolicy Delete
- DeletePolicy retain = specify res. to preserve in case of CF deletions
- DeletePolicy Snapshot = take a last snapshot before del.

CF Stack policies:  
JSON doc that defines the update actions that are allowed on specific res. during stack update.

CF Custom resources:  
define resources not yet supported, custom resources from 3rd party. They are integrations backed by lambda functions

CF stack sets:  
CRUD stacks across multiple accounts and regions with a single operation, the developer can deploy the same CloudFormation stack to   
multiple regions without additional application code. ex: for geographic load testing of an API.

CF Dynamic references:  
Reference external vals stored in System Manager, Parameter Store and Secrets Manager within Cloud Formation templates.   
for ex. get the RDS DB instance master password from secret manager. 

CF Template settings:  
- Mappings section matches a key to a corresponding set of named values: ex if you want to set values based on region
- Metadata section to include arbitrary JSON or YAML objects thatprovide details about the template.
- Parameters enable you to input custom values to your template each time you create or update a stack.
- Conditions section contains statements that define the circumstances under which entities are created or configured

CF Deploy a Lambda 2 options:  
- write the code directly inside the CloudFormation template in the json
- Upload a ZIP file containing the function code to Amazon S3, then add a reference to it in an AWS::Lambda::Function

CF pseudo-parameter:  
predefined values by AWS CF:  
AWS::AccountId  
AWS::StackName  
AWS::Region <--  
ecc...  

# SQS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/986a4ac4-0e84-4b1a-8731-f06774f3b7ae">

### Simple queueing service
Producer sends message to SQS, like with SDK (sendMessage API)  
the message remains until it gets consumed and deleted (def. 4 days max 14 days)  
it can have duplicated messages  

A Consumer polls messages and processes it and deletes it from queue  
Fully managed service used for decoupling apps. max 256kb message. running on EC2 or lambda or your local server  
max 10 messages at a time. after consuming delete the message so no other consumer uses it.  
Can have multiple consumers at a time. we can horizontally scale by adding more consumers. At least once delivery because queue can have duplicate messages

We can enable Auto Scaling, the queue has an ApproximateNumberOfMessages which can be increased by setting a CloudWatch alarm that increases it

SQS has unlimited nr of messages and unlimited throughput, so we can scale fronted and backend independently and decouple them  
<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/59d0b910-d7e0-4381-9a51-42c20d4fcf22">

SQS security:  
inflight encryption with HTTPS API  
- at rest encryption with KMS keys
- client-side if user handles it himself
- AMI policies to regulate access
- SQS access policies (if you want cross account access or publish S3 event notifications)

SQS Message Visibility Timeout:  
After a message is polled by a consumer it becomes invisible to other consumers, it has 30s to process it,  
if the message hasn't been deleted cuz not processed yet, it will be put back in the queue and be processed twice, if u need more than 30s to process it  
use the "ChangeMessageVisibility" API to change the timeout time to increase the value of VisibilityTimeout

SQS Dead Letter queues (DLQ):  
We can set a threshold to how many times a message can go back into the queue because a consumer fails to process it, after the "MaximumReceives" threshold  
is exceeded it goes in the deal letter queue. DLQ useful for debugging, messages in there expire after the retention period, so set a higher 14days for the DLQ  
Debug the DLQ, fix your code, use the "Redrive to Source" to put the DLQ messages back in the Source queue so it can be processed correctly this time

SQS Delay queue:  
Delay a messages so cosumers dont see it immediately, up to 15 min, or per message individually, default is 0

SQS Short polling:  
Querying only a subset of its servers to determine whether any messages are available for a response.

SQS Long polling:  
When a consumer requests a message from the queue it can wait for messages from queue to arrive if there are none in the queue, this is called long polling  
long polling decreases the number of API calls, messages get processed at the end of the wait period, higher efficiency  
the wait time can be 1-20s, its better than short polling. IT RETURNS MESSAGES AS SOON AS THEY BECOME AVAILABLE:
to enable: Set the ReceiveMessage API with a WaitTimeSeconds of 1-20s

SQS Extended client library ECL:  
library that uses an s3 bucket to send messages higher than 256kb, the queue sends a message telling the consumer to get the larger message from the s3

Must know API:  
- CreateQUeue, Delete queue  
- PurgeQUeue, deletes all messages  
- SendMEssage, RecieveMessage, DeleteMessage  
- MaxNumberOfMessages (for receiving) def. 1 max 10  
- ReceiveMessageWaitTimeSeconds: Long Polling  
- ChangeMessageVisibility: change message timeout  

FIFO queue:  
Deduplication methods: if you send the same message twice within 5 minutes it gets discarded, BY USING "MessageDeduplicationId" PARAMETER.
- Content-body SHA256, if 2 are found it discards
- Explicit-body, if 2 messages with explicit body are found it gets discarded

FIFO queue Message grouping:  
We can give the queue messages a MessageGroupID, so it gets in the same queue but gets processed by different consumers,   
USED TO ORDER THE MESSAGES, SO EACH MESSAGE IS PROCESSED IN THE ORDER IT IS RECEIVED  
so we can have type A, B, C ecc... messages and each one type gets processed by different consumers  

# SNS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/ffc2f900-e207-4a4f-b527-f5c8ac5ab1f0">

### Simple notification system
if u want to send the message to many receivers at the same time, email, SQS queue, shipping service ecc....  
It uses the pub/sub method. A buying service publishes the message inside and SNS topic, and subs of the topic listen to the SNS topic notification  
subs can be Lambda, email, SMS, Kinesis data stream, SQS, HTTP endpoints, and many services


SNS security same as SQS:  
inflight encryption with HTTPS API
- at rest encryption with KMS keys
- client-side if user handles it himself
- AMI policies to regulate access
- SNS access policies (if you want cross account access or publish S3 event notifications)

Fan Out pattern:  
push the same message to multiple SQS by pushing it only once. SQS are a sub of SNS  
ex. for S3 events to multiple queues.

SNS message filtering:  
JSON policy to filter messages and only send the messages of a filter in a specific SQS queue.

# Kinesis

Easy to collect/process/analyze streaming data in real-time, such as Application logs, metrics, website clickstreams, IoT

- Kinesis Data Streams: capture, process and store data streams
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5c8f258e-1e14-4a5f-95a5-685f309db280">  

is a way to stream big data in your systems (ingest data at scale), is made of multiple "Shards" per stream (can be used to scale up or down),
  data will be split to Client. it sends a record to the stream that is made out of Partition Key and Data Blob (max 1MB)
  after it reaches KDS it gets send a consumer, with the Partition Key, Data Blob, and Sequence Nr.
  Partition key hashed a unique ID you pass, and it always sends the data from that provider to the same shard. Multiple hashes from multiple
  sources can be sent to the same stream, so it's distributed. 
  - Can give an ORDER by using "SequenceNumberForOrdering"
  - To handle DUPLICATES, you can include a unique ID in each record that you write to the stream, and check if it
    already exists in DynamoDB when consumer tries to process it. 
  - Retention of messages 1-365 days (by default 24hrs, if app takes more to process it gets lost)
  - because of this: ability to reprocess data
  - once data is inserted in Kinesis it can't be deleted (immutability)
  - Capacity modes: prevision mode: fixed nr of shards, scale manually with API, you pay per shard provisioned per hour
  - Capacity modes: on-demand mode no need per provision or manage the capacity, scale based on throughput during the last 30 days, pay per stream per hour
  - Security: encryption at rest (AWS KMS customer master key (CMK)), in flight with HTTPS, IAM policies, VPC endpoints
  Producers: SDK, Kinesis Producer Library KPL, Kinesis Agent, monitor log files.
  ProvisionedThroughputExceeded: too many inputs in a shard, solution: use highly distributed partition key, retries with exponential backoff, increase shards

Consumers:  
custom with SDK, Kinesis Client Library, Lambda ecc... can have multiple consumers getting from the same shards.  
Shared fan out consumer: max 2mb/s split between all consumers of a shard (pool model)  
Enhanced fan-out Consumer: 2mb/s per consumer per shard by subscribing (push model to subscribers)  

Kinesis Producer Library (KPL):  
simplifies producer application development, allowing developers to  
achieve high write throughput to a Kinesis data stream. helps you write to a Kinesis data stream.  

Kinesis Client Library (KLC):  
Java library that helps read records from Kinesis Data Stream with distributed apps. sharing the read workload.  
Each shard is to be read by only one KCL instance, 4 shards = max 4KCL instances, 8 = 8 instances  

Kinesis operations:
- shard splitting: increase the stream capacity: used to divide "Hot shards"
- Merging shards: decrease capacity and save costs by merging them. both used to scale up and down kinesis

Kinesis Adapter:  
recommended way to consume streams from DynamoDB for real-time processing.  

Kinesis Data Firehouse:  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/41492d0c-faaa-4e15-8d66-c1bd21b45c8c">  

  Managed (ingest data at scale) used to load data into specific services, streams into ASW data stores near real-time, fully managed serverless.  
  Takes data from producers / kinesis data stream. And Batch writes data into destinations to:  
  (to know)(S3, Redshift, Amazon Open Search, 3rd party destinations, HTTP endpoints) custom destinations (cant process just load data)  
  it can first transform the data by compressing/converting ecc.. or custom with Lambda  
  it has auto scaling not like KDS, there is no storage so no replay capability.  
  
- Kinesis Data Analytics:
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/fb8c6601-9e33-4380-b85e-e160cb48ef3d">

  analyze data Streams for SQL, fully managed.  
  It can read either from KDS or KDF, and apply SQL statements to perform real-time analytics + can add s3 to join data  
  it can then send the data to KDS (that can do real-time processing of the data) or KDF ( that can send it to S3, Redshift, other)  

  Analyze data for Apache Flink:  
  For managed clusters, it can read data from multiple sources at a time  

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a364a0e5-ad38-4286-9a42-7f7bf9491009">

Monitoring / troubleshooting / logs  

# CloudWatch
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/b5813ebd-7021-4505-abbf-ec18af030d43">

Collect and track metrics  
collect, monitor and analyze and store Logs  
Send notifications for Events  
- Metrics:
  for every service in AWS, it's a variable to monitor ex CPU, networking, they have timestamps and you can create dashboards with them
  EC2 instance metrics every 5 min, there is a "detailed monitoring" for a cost you get every 1 minute. By default no log from EC2,  
- need to setup an "agent" to push them. "CloudWatch Unified Agent" allows you to get more granular extra data from EC2   instead of the default ones
- If you need granularity higher than detailed monitoring which is 1min, use High resolution, with data at a granularity
  of 1 second, 5 seconds, 10 seconds, 30 seconds, or any multiple of 60 seconds.
- You can define Custom Metrics, using the API PutMetricData
  - The "--dimensions" parameter further clarifies what the metric is and what data it
    stores. You can have up to 10 dimensions in one metric, and each dimension is defined by a name and value pair

  Metric filters define the terms and patterns to look for in log data as it is sent to CloudWatch Logs, then it turns metric filters log data into numerical  
  CloudWatch metrics that you can graph or set an alarm on.
- Logs:
  - log groups: name representing the application
  - log stream, instance within application
  - log expiration policy
  - can send logs (export) to multiple sources. Either using S3 Export, export in batch when running the API CreateExportTastk Or
    Using "CloudWatch log subscription" real-time logs 
  - logs encrypted by default, you can use your own KMS
  - "CloudWatch Logs Insight" used to query the logs
  - "CloudWatch metric filter" filter expressions (ex. count "ERROR" in logs) to trigger alarms
  - "CloudWatch Logs agent" collect local logs from the instances and upload them to CloudWatch 
- Alarms:
  used to trigger notifications. status OK ISUFFICIENT_DATA ALARM
  it can do 3 things
  - Stop / terminate / reboot / recover EC2
  - Trigger Auto Scaling
  - Send SNS notification (here we can attach lambda and do anything)
  we can use Composite alarms to monitor multiple metrics at a time, useful to filter "alarm noise"

There are two types of API logging in CloudWatch: execution logging and access logging.  

Namespace is a container for CloudWatch metrics. Metrics in different namespaces are isolated from each other, so that  
metrics from different applications are not mistakenly aggregated into the same statistics. They are like repositories,
each namespace = different Cloudwatch with unique metric  

Cloudwatch Synthetics scripts that monitor APIs, URLs, Website workflow ecc... and check the availability and correct operation of these elements  
reproduces what customers do on a website for example, access to Headless chrome  

Logs encryption:  
you can encrypt logs with KMS keys, is enabled at the log group level, by associating a CMK with a log group

CloudWatch Evidently:  
Safely validate features by serving them only to a small % of users. Use feature flags to test a feature only by some users. 

# EventBridge
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/c3fd578a-6ee7-4551-9f57-7404a8375f8e">

- schedule scripts / cron jobs in the cloud
- react to event pattern ex: IAm user signing in -> send email with SNS, it can also trigger lambda, step function, kinesis ecc..,
  or if CodeBuild Fails, if an EC2 is started, if S3 upload ecc...  
It's the default event bus for AWS services, it can react to events from 3rd party "Partner Event bus", or custom Apps "Custom Event Bus"  
You can archive events or replay archived events.

eventbridge event buses in one (target) account can be a target of another event rule in a source account.

Resource-based Policy can deny events from other AWS acc, or aggregate from multiple acc. into one using "Multi-Account aggregation"
  
# X-Ray
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/250aee0d-120c-4f36-ae9f-2b3b9136beb4">

It has a Visual analysis of your application  
Troubleshoot app. performance and errors  
You can understand dependencies between microservices  
understand where the error is  
It uses tracing, each component dealing with the request adds its own "trace"  
IAM, KMS  
To enable it:  
modify your code using x-ray SDK then install the xRay daemon or enable AWS integration .ebextensions/xray-daemon.config file to the source code to enable the X-Ray daemon (FOR BEANSTALK ONLY)  
The X-Ray daemon listens for traffic on UDP port 2000  
You can only use X-ray with Fargate as a "SIDE CAR" because there is not EC2 image  

How to instrument your code: instrument = measure the product performance, diagnose errors and write trace information. you use the X-Ray SDK.  
- Segments: each app. will send them.
- Trace: segments collected together to form an end-to-end trace.
- Sampling: decrease the amount of requests sent to X-ray. control the amount of data.
- Metadata tags to record additional data that you want stored in the trace but don't need to use with search.
- Annotations in AWS X-Ray that can be used with Filter expressions
- Subsegments provide more granular timing information and details about downstream calls that your application made to fulfill the original request.
  - !! namespace "AWS" for AWS SDK calls; "REMOTE" for other downstream calls.

"GetTraceSummaries":
you can search for segments, get the list of trace IDs of the application    

"BatchGetTraces":  
retrieve the list of traces

AWS Distro for OpenTelemetry:  
AWS-supported distro of the open-source project Open Telemetry  
Collects distributes traces and metrics from your apps. It's like xray but open source.

X-RAY SDK Variables:  
- _X_AMZN_TRACE_ID: trace ID, and parent segment ID
- AWS_XRAY_CONTEXT_MISSING: behavior in the event that your function tries to record X-Ray data, but header is not available.
- AWS_XRAY_DAEMON_ADDRESS: exposes the X-Ray daemon's address  

# CloudTrail
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f112834f-04ad-4f4c-b183-937edb7bd2b7">

Internal monitoring of API calls being made  
Audit changes to AWS resources by users.  
History of all events like who deleted something, can put logs into CloudWatch Logs or S3  
3 types of events:
- management events: anything that modifies your res. or aws account (enabled by default)
- data events: S3 object-level activity ex: S3 GetObject, DeleteObject ecc... or AWS lambda execution activity (all not enabled by default) 
- Insights events: because of many logs and types, it automatically analizes your logs and detects unusual activity / patterns based on your usual baseline
event retention: stored for 90 days, if u want longer, send to S3 and use Athena to analyze them.

You can have Event Bridge integration ex. User changes IAM Role -> CloudTrail -> triggers an event -> EventBridge -> SNS email

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/30e0f21a-9804-461c-8a8d-6a1864818194">

# Lambda
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/eba57b02-7f3d-479b-adc4-f2e0feb4e258">

Serverless: just deploy code without provisioning / managing servers.  
- Virtual function no server to manage  
- max 15min execution  
- runs on-demand when it gets invoked  
- scaling is automated  
- pay just for request and compute time  
- Burst Concurrency Limits: 500-3000 (default 1000) If you invoke the function again while   
- the first event is being processed  

Main integrations:  
- API Gateway: to create Rest API  
- Kinesis: data transformation on the fly  
- DynamoDB and S3: triggers for when something happens in DB  
- CloudFront: lambda edge  
- CloudWatch events / EventBridge: when we want to react when something happens in our structure, or if we want to use a CRON eventBridge rule or Create an Amazon CloudWatch Events rule that is scheduled to run every x minutes.    
When your resources change state, they automatically send events into an event stream. You can create rules that match selected events in the stream and route  
them to your AWS Lambda function to take action. Like ex: associate Lambda function with CodePipeline  

CloudWatch Logs: stream the logs where we want  
- SNS: React to notification  
- SQS: process queue messages  
- Cognito: react for example to logins  
ALB: HTTP client call to ALB -> ALB converts HTTP to JSON request and invokes Lambda synchronously  
<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/858dd518-f375-44a1-bd80-0a1c2abc0613">

<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5976dd42-d6d8-4e95-aa0b-433917a1184b">

Ways to process data:  
- Synchronous invocation: happens when using CLI, SDK, API Gateway, ALB, Cognito, step functions, CloudFront (all user-invoked or service-invoked services)
  it means you are waiting for the result and the result will be returned to you, errors must be handled client side ex button to retry

- Asynchronous invocation: S3, SNS, CloudWatch events ecc.. ASYNCHRONOUS = Invoke the Lambda and get a response immediately.
FROM API GATEWAY "X-Amz-Invocation-Type" header pecifies the invocation type of the Lambda function, and setting it to 'Event' means that the function will be invoked asynchronously  
The events are placed in an EventQueue for the lambda to consume, if the vent fails it retries it 3 times, after 1 min and after 2 min  
which means your code needs to be Idempotent so it shouldn't break if it retries. We can define a DLQ dead letter queue with SNS or SQS for failed processes  
strategy to do this:  
Extract the value of a unique attribute of the input event. (For example, a transaction or purchase ID.)  
2.Check if the attribute value exists in a control database. (For example, an Amazon DynamoDB table.)  
3.Depending on the results, complete the following step:  
If a unique value exists, then end the action without producing an error.  
-or-   
If a unique value doesn't exist, then proceed with the action that you originally designed.  
So get ID, check DynamoDB if duplicate, act accordingly  

- Event Source Mapping (important): Kinesis, SQS, DynamoDB, Lambda needs to poll from the source, it doesn't get invoked. Process streams or Queues (SQS)
  An event-source mapping must be created on the Lambda side to associate the stream with the Lambda function

Event: JSON contains data the function is going to process  
Context: methods and properties that provide information about the invocation, function, and environment LIKE REQUEST ID 
<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/6344ebcd-3561-4f2a-aae4-1423c294bdb7">

Destinations: Send the results of successful or fails to a destination (Lambda, SNS, SQS, EventBridge)  

Lambda Execution Role (IAM Role) Policy to AWS services to ex. event source mapping, permission to Lambda to read from services / write to resources  
Lambda Resource base policies: To give other accounts and AWS services to use and invoke Lambda resources  

EnvironmentVariables: Key/value pairs in String form, adjust the function behavior without updating code. Like a config file.  
features to customize how environment variables are encrypted:  
- Key configuration: On a per-function basis, configure Lambda to use an encryption key that you create (CMKs),
  if you dont config lambda creates automatically Lambda uses an AWS managed CMK
- Encryption helpers: The Lambda console lets you encrypt environment variable values client side, before sending
  them (preventing secrets from being displayed unencrypted in the Lambda console, or Lambda API)

Logging and monitoring:  
CloudWatch logs: lambda logs are stored there if function has the policy, Lambda functions automatically store logs generated by code in CloudWatch Logs. However, 
the Lambda functions require the execution role to be configured with the appropriate permissions  
CloudWatch metrics: lambda metrics are displayed there, error counts, invocations, success rates...  
X-Ray: Can use in lambda if correct IAM role and enable "Active Tracing", it will run the xray daemon for you  

Edge Functions: code attached to CloudFront execution, logic at the edge locations, runs close to users to minimize latency. Deployed globally, serverless  
used for: website security, SEO, intelligent Route, real-time image transformation, user authentication, user prioritizing, user tracking and analytics...  
- CloudFront Functions: JS functions for high scale, latency-sensitive CDN customization. used to customize the viewer response / requests (before CloudFront
  forwards the request to the origin / after it gets the response and returns it to the client). Ex for header manipulation, URL redirects...
- Lambda@Edge: Used to change CloudFront request / responses to both Origin and Viewer requests. higher package dimensions but slower. do what CloudFront functions
  do but with file system access, handles heavier data so it can use the body of the HTTP request, longer execution, both for viewer and origin.
  IT CAN AUTHENTICATE / AUTHORIZE Users for PREMIUM pay-wall content.

Lambda are launched in a VPC outside of yours, in a VPC managed by AWS so it can't access your VPC resources.  
But you can deploy it in your own VPC using AWSLambdaVPCAccessExecutionRole  
Lambda in a public subnet doesn't have access to internet or public IP, if you want you need to:  
By default it doesn't have access to the internet in your private VPC, so you need to give it access with NAT Gateway / instance, YOU CANT CONNECT IT DIRECTLY TO  
INTERNET GATEWAY.
Or you can use a VPC endpoint to privately access AWS services without NAT.  
<img width="300" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/489bec1f-22e1-4e14-9e26-058ed41dc987">

Lambda function configuration:  
Ram from 128MB to 10GB, if you need to increase vCPU you NEED to increase RAM, so if your app is CPU-bound, increase RAM!  
Timeout: default 3s, max 15min, if you need more use Fargate, EC2, ECS  
Execution Context: temporary runtime that initializer external dependencies / services connection ecc.. is maintained for a short period expecting a new invocation ex:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/b8e7144f-9bdb-4baa-bd52-29d693aa81f5">

/tmp folder if your function needs to download a big file, max 10GB of disk space.  
if you need persistence space, use S3.  
Data is SAVED in this directory, persists across these retained invocations, making it useful for caching.  


Lambda Layers:  
custom runtimes for not supported languages, or create a layer to externalize dependencies to reuse them without having to repackage them to deploy, because the lambda can  
reference these layers. ONYL FOR LIBRARIES, NOT DB keys ecc... use ENV. Variables for those.

File System Mounting:  
Can access EFS if running in VPC, it's a config to mount EFS file system to local directory at initialization, it uses EFS access points.

Concurrency:  
the more we invoke the more executions up to 1000, we can set a "reserved concurrency" to limit the max amount per each function.  
if we don't set limit, we can have 1 function taking up all 1000 executions, and have the other functions not being able to start.
!! Lambda will keep the unreserved concurrency pool at a minimum of 100 concurrent executions, so that functions that do not have specific limits set can still process requests. So, in practice, if your total account limit is 1000, you are limited to allocating 900 to individual functions. So if you have 2 functions that need to have the same concurrency, you can max do 450 and 450.

Cold start and provision concurrency:  
new instance so all initializations have to start. The first request has a higher latency,  
you can use a "provision concurrency" to allocate before the function is invoked in advance so cold start never happens.  

Dependencies:  
to include them, you need to zip dependencies and code all together, if > 50 MB upload in S3 and reference it.  

Lambda and CloudFormation:  
inline directly in the CF code for very simple functions, we can't include external dependencies.  
Through s3, store the Zip in S3 and refer it in CloudFormation code. (BOTH IN THE "ZIPFILE" PARAMETER in the template)

Lambda Container Images:  
Deploy Lambda functions as Container images up to 10BG, pack complex and large dependencies in a container, instead of Zip and upload.

Lambda Versions:  
When you work on a function we work on $LATESTS, we publish we create a version which is immutable and can't be modified (v1,v2 ec...) = code + config.

Lambda Aliases:   
used to give the user a stable endpoint, "dev" "prod" ecc... point to the right lambda version. We can also use them for Canary deployment,   
ability to return to older versions of the function quickly. A Lambda alias is a pointer to a function version that you can update.

Lambda and CodeDeploy:  
help automate the traffic shift for Lamba aliases.  
CANNOT USE IN-PLACE DEPLOYMENT (just upgrade the code), MUST USE ONE OF THESE, because instances must be stopped and  
restarted with the new version, only EC2/On-Premises can do it.  
- Linear: glow traffic by x percent every n minutes  
- Canary: x percent then 100%  
- AllAtOnce: Immediate

Lambda function URL:  
to expose the function as HTTP endpoint without having to use API gateway ecc... can create a unique URL endpoint, access only with public internet  
can have Resource-based policy, CORS security like S3. we can set AuthTypeNone or AuthTypeAWS_IAM for access.

Lambda and CodeGuru: can be activated from console to get runtime performance insights

Lamba Limits per region:  
128mb-10gb RAM allocation  
max execution 15min  
env var. max 4kb  
disk capacity /tmp 512MB to 10GB  
concurrency executions: 1000 (can be increased by support)  
compressed zip deploy: 50MB  
size of uncompressed: 250MB  

best practices:  
never have recursive code.  
Heavy-duty work put it outside the function handler:  
connection to databases  
initialization of AWS SDK  
pull in dependencies  
Use env. variables for stuff that changes frequently:  
DB connection strings  
S3 bucket ecc....  
Passwords, encrypted with KMS  

Lambda all console.log() statements are automatically sent to CloudWatch.  

You can Invoke functions with test events, also events can be shared between IAM users and be editable.  

# DynamoDB
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/60590f72-b4e5-4b77-b546-237a679894c6">

NoSQL serverless DB  
They can scale horizontally for more Read/Write capacity, do not support joins or operations like "SUM", "AVG"  
it a fully managed highly available with replicas across multiple AZ.   
Scales massive workloads  
Fast and consistent performance  
Low-cost and Auto-Scaling  
Standart and IA calsses.  

Each table has a Primary Key, infinite rows, each one has attributes can be null or added over time  

options to choose the primary key  
- Hash: unique for each one like "user_id"
- Partition Key + sort key: "user_id" + "game_id" unique for each item. For example each user can attend multiple games
  so the same partition key is equal in 2 rows but different game_id keys, the combination is unique

Control the R/W capacity modes:  
- Provisioned mode (default): specify the r/w per second you need to plan your capacity beforehand, you pay for what is going to be provisioned
  - RCU read capacity units: represent one strongly consistent read per second or 2 eventually consistent read per second, for an item up to 4KB
    - read eventually consistently: if we read right after a write data could be old cuz it hasn't replicated you from the copy we are reading from
    - Eventually consistanc consume Half the RCUs, divide by 2
    ex: 10 strongly consistent reads per second with item 4KB = 10RCUs
    ex: 16 eventually consistent read per second with item size 12KB = (16/2) * (12/4) = 24RCUs
    ex: 10 strongy consistant reads per second with item size 6 KB = 10 * (8/4) because 6 gets rounded to 8KB
  - WCU write capacity units: are the throughput, represent one write per second for an item up to 1KB, if larger than 1KB, more WCU
    ex: we write 10 items per second with item size 2KB = 20WCUs
    ex: we write 6 items per second, with item size 4.5KB = 30WCUs (the 4.5 gets rounded up to 5)
    ex: 120 items per MINUTE with item size 2KB = 120/60 * 2KB = 4WCUs
  - option to enable auto-scaling to meet demand
  -  throughput can be exceeded temporarily by using "burst capacity"
    
- On-demand, no provision pay for what you use but more expensive

Data is stored in partitions:  
primary keys go through hash to know which partition they go to, WCU and RCU are going to be evenly spread across partitions

Throttling:  
reasons: Hot keys (one partition read too many times ex popular items), Hot partitions, very large items, the solution is Exponential backoff,   
hot partition due to the order date being used as the partition key and this is causing writes to be  
throttled. solution to ensure the writes are more evenly distributed in this scenario is to add a random  
number suffix to the partition key values.  
distribute partition keys, if RCU issue use DynamoDB Accelerator DAX  

Limit parameter:  
To control the amount of data returned per request, Limit parameter. Helps prevent situations where one  
worker consumes all of the provisioned throughput, at the expense of all other workers.  

Basic Operations:
Writing Data:
- PutItem: create a new item or fully replace an old one (same primary key)
- UpdateItem: used to edit a few attributes, but it can also edit all of them, it inserts a new item if not exist
- Concurrent Writes: the second one overwrites the first one
- Conditional Writes: accept a Write / update /delete only if conditions are met
  attribute_exist / attribute_not_exist / attribute_type / contains / begins_with / IN and between / size (2 concurrent write could fail because 1 updates and the other
  finds the new data and doesn't match so the second one fails)
- atomic writes: one says increase by 1, the other increase by 2, at the end the result is 3, both writes succeed
- batch writes: Write / update many items at a time
- TransactWriteitems: multiple items Transactional all-or-nothing write.
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/d5a35a98-dbf1-4cac-a847-e70c3bfcb9bb">  
 
Reading Data: 
- GetItem: read based on Primary Key (hash / hash + range), eventually consistent (default) / strongly,
- ProjectionExpression to read only certain attributes
- Query: Based on Partition key value (required) , and optional sort key values (=,<,> ec...), FilterExpression additional filtering after query for non-key attributes
- Scan: get the entire table and filter out on your application (inefficient).
  supports parallel scans for faster results.
  To minimize the impact of the scan:
  - on the provisioned throughput Set a smaller page size for the scan
  - Optimize the table to use query instead of scans for big tables.
 
Deleting Data:
- Delete an individual item
- conditional delete
- DeleteTable
 
Table Cleanup:
- scan + deleteItem very slow and expensive
- drop table + recreate table fast and cheap

Copy DynamoDbTable:
- use AWS Data Pipeline
- Backup and restore into a new table
- Scan + PutItem 

 You can batch operations to reduce API calls, it will be done in parallel for efficiency, it can fail if throttling  
 and if the table doesnt have enought Read/Write Capacity
 - BatchWriteItem: up to 25 PutItem / DeleteItem
 - BatchGetItem: up to 100 items, max 16MB.

PartiQL:  
SQL-compatible query (no joins) if all you know is SQL. You can select, CRUD, and supports batch operations.  

Local Secondary Index LSI:  
Alternative sort key for table (same as partition key) based on sort key = scalar attribute (String, Number, Binary) + up to 5 local secondary indexes per table  
must be defined at table creation time. !! 
ex: give me all the games that have been played by this user between 2021 and 2023. We have a different sort key thanks to LSI  
With a local secondary index you can have a different sort key but the partition key is the same.  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4cb1c79c-c587-4ab8-b635-f573ac1fd9c7">  

Global secondary index (GSI):  
Alternative Primary Key (hash / hash+range)  
Contains a selection of attributes from the base table, but they are organized by a primary key that is different from that of the table.  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/01541488-4980-4905-8c41-e73febf490a8">  

Optimistic Locking:  
"conditional writes" ensure the item hasn't changed before you update/delete it, each item has an attribute that acts as a version number.  
read a record, take note of a version, check that the version hasn't changed before you write the record, Or abort.  
NO APPLICATION PERFORMANCE LOSS. (you dont need to maintain the connection alive always)    

Pessimistic Locking:  
lock the record for your exclusive use until you have finished with it. This can prevent certain users from reading, updating.   

Atomic counter:  
Numeric attribute that is incremented when UpdateItem is called, is not accurate because even if the update fails it increments, used for example for users on a website  
or counts that can have slight miscalcumations.  

DynamoDB DAX: 
Fully managed highly aval. seamless i-memory cache for DynamoDB, fully secure, microseconds cached reads and queries. Doesnt require   
application logic modification because compatible with existing DynamoDB APIs. Solves the "hot key" problem so too many reads.  

DynamoDB Streams: 
ordered stream of item level modifications (CRUD), streams can be sent to kinesis, Lamba.   
use cases: react to changes in real-time ex: send welcome email to users, analytics, insert in derivate tables  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/1f266a95-fd6f-474a-ab9a-2ca2036e5ea4">  
With lambda for table -> DynamoDB Stream -> AWS Lambda event mapping source polls from it

DynamoDB TTL: time to live delete item after an expiry timestamp. Ex to delete session data. It takes up to 48hrs after the TTL for it to get deleted. Or deleted after the user logged out of the system.  

DynamoDB Transactions: all-or-nothing operations. THE QUERY IS TRANSACTIONAL NOT THE TABLE, You dont need to update the table  
In read mode it can be consistent/strong consistent/"transactional"  
in write mode it can be standard/"transactional"  
consumes 2x WCUS and RCU  
TransactGetItem one or more GetItem operations  
TranctWriteItem ore or more PutItem, UpdateItem, DeleteItem  
ex: multiplayer games, financial  
ex: 3 transactional writes per second with 5KB = 3 * 5 * 2 = 30WCU  
ex: 5 transaction reads per second with 5KB = 5 * (8/4)(5 got rounded to 8) * 2 = 20RCU   

DynamoDB Session State cache:  
DynamoDB can store sessions is serverless, also ElastiCache can but it's in memory.   
At exam if "Session state STORE in memory" = elastiCache, it it talks about automatic scaling = DynamoDB. Also EFS can do the exact same thing but it's a file system not DB,  
EBS and Instance Store can't be used for shared caching, only local. S3 could be but it has high latency not meant for that small objects  

Write sharding:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/ba8c96fd-4345-459a-92a9-29b125f3344f">  

DynamoDB large objects:  
max allowed in Dynamo 400kb, we use an s3 bucket to contain a large object  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4e0d48d0-07df-4c6f-9a3b-949edae5cbd5">  
OR  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5e20f558-9a14-4fb9-9972-01e02c68b8ff">

Security:  
VPC Endpoints to access without internet  
Access with IAM  
encryption KMS, in transit SSL/TLS  
backup and restore features  
Global tables, fully replicated multi-region  
DynamoDB Local dev and test apps locally without the official server  
DMS  
Can use identity providers / Cognito to exchange credentials for temporary credentials with roles, and can do operations only on data they own  

Enabling DynamoDB Streams on the table allows you to capture and process changes (inserts, updates, deletes) to the table in real-time  
StreamEnabled — Specifies whether a stream is enabled (true) or disabled (false) for the table.  
StreamViewType — Specifies the information that will be written to the stream whenever data in the table is modified:  
- KEYS_ONLY — Only the key attributes of the modified item.
- NEW_IMAGE — The entire item, as it appears after it was modified.
- OLD_IMAGE — The entire item, as it appeared before it was modified.
- NEW_AND_OLD_IMAGES — Both the new and the old images of the item

Permissions to allow uaccess to CERTAIN items and attributes:  
Restrict access to specific items based on certain primary key values  
identity-based policy that restricts access TO AN ENTIRE TABLE, not specific items  

# API Gateway
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/e272307f-1bfe-4156-8f98-dbba2b4c4cd8">

Serverless Expose functions to the world / Access to the app with REST APIs, even with Authentication  
It integrates with / can call:   
- Lambda function: invoke / expose the REST API
- HTTP: HTTP endpoints even on-premise / ABL (HTTP API)
- AWS Services: Any AWS service API with gateway (HTTP API)
- Websockets: two-way interactive communicatio. between browser and server, enable stateful apps. server can push info to client
  used in real-time apps like chat, collab platforms, multiplayer games.
  Client connects to API Gateway, it gives a Connection ID, lambda uses logic and stores in DB the connection ID,
  it uses a Connection URL Callback to send data to client. You can use Routing to reroute to a specific backend when
  the connection is open (for example to do CRUD different backends)

In AWS API Gateway, API keys by themselves do not grant access to execute an API.  
They need to be associated with a usage plan, and that usage plan then determines which API stages and methods the API key can access. added to a usage plan by calling the "CreateUsagePlanKey" method!

Mapping templates: ex: You can transform the incoming JSON into a valid XML message for the SOAP backend interface using mapping templates  

3 ways to deploy (called Endpoint Types)  
- Edge-Optimized (default): for global clients,  Requests are routed through CloudFront Edge locations (improve latency)
  the API Gateway still lives in only one region
- Regional: for clients in the same region, could manually combine with CloudFront
- Private: Can only be accessed from your VPC using an interface VPC endpoint (ENI)

Security:   
Can Authenticate users with:  
- IAM roles (for INTERNAL apps) create and IAM policy auth and attach to User/Role, IAM credentials are signed
  in the headers.
- Resource policies: Allow for CROSS ACCOUNT access combined with IAM security, or for specific IP Addresses, or VPC
  endpoint.
- Cognito user pools(for external users) users sign in with Congnito first which provides token
- Custom Authorizer (Lambda Authorizer) Token-Basesd auth. the lambda if auth is correct returns an IAM policy that will
  be cached. if you want to implement a custom authorization scheme that uses a bearer token authentication
  strategy such as OAuth or SAML, or that uses request parameters to determine the caller's identity. (TOKEN authorizer e REQUEST authorizer)

- Custom Domain Name HTTPS: security through integration with AWS Certificate Manager (ACM)
  
Deployment Stages:  
After Making a change must be deployed it or else changes wont apply, its done through "stages"  
each stage has its own config. parameters. Stages are kept and can be rolled back if needed  
<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/8baded8a-55ea-461e-8980-df53eb2cebc8">  
Update stage variable value from the stage name of test to that of prod: Update stage variable value from the stage  
name of test to that of prod  

Stage variables: like Env. variables (config) but for API Gateway, use them for often changing values  
so it prevents redeployment every time. 
- A common use is defining to which Lambda function it points and when you deploy V2 change a % of traffic to the new one changing the Stage Var. (Canary Deployment)  
- Also used to implement and run different versions for testing purposes ex: alpha .tutorialsdojo .com endpoint and beta release through the beta .tutorialsdojo .com endpoint

Integration types:  
- Mock: API Gateway returns a response without sending the request to the backend
- HTTP / AWS (lambda or AWS services): You must config the integration request and response, setup data mapping
  with mapping templates for request and response which means we can change the data before it reaches client or backend.
  Mapping template: modify parameters and strings, filter output results, use VTL scripting language for loops,
  if ecc.. We can use it to convert from JSON (REST APIs) to XML (SOAP APIs) 
- AWS_PROXY (lambda proxy) incoming requests from the client is input data to lambda without changing the data,
  the function handles all logic
- HTTP_PROXY Same as AWS_PROXY, requests passed directly to the backend, without modification, for API
  or HTTP endpoints.

It has integration with Open API spec. (it's a common way to define REST APIs, using API defined as code.  

Can config API Gateway to perform validation so if the data structure doesn't correspond to a schema returns 400-error  
(reduces nr of calls made)

Caching API responses:  
reduces the nr of calls made, default TTL is 300s, min 0 max 1hr  
Caches are different for every stage. We can invalidate the Cache immediately and flush it, or the user can do that  
with "max-age=0" if it has the IAM Auth.  
We can CHOOSE to Cache only a type of request/response for example only POST  
Ticking the "Require authorization" checkbox ensures that not every client can invalidate the API cache  

You can make your API available for money to customers. We can config how much and fast they can access them, how many calls  
possible, use API Keys to identify them, callers pass the key in x-api-key header.  
Throttling default 10000 rps across all API, in case it happens Error 429 too many req.  
Can set Stage limit and Method limit to increase performance  

CloudWatch Logs: Log contains info about request/response body  

X-Ray: enable tracing to get extra info  

CloudWatch Metrics: Metrics based on stages, CacheHitCount & CacheMissCount, Count (nr of req), IntegrationLatency  
(time between request and response from the backend), Latency (time between request and sending response to user),   
4XXError (client side) 5XXError (server side)  

CORS: must be enabled if you want to receive calls from another domain  

# CICD 

Automate and Push code without doing anything else  
we want to push code in a repo and deploy it to AWS.

CI: Continuous integration, devs will push code to repo, a testing / build checks code, the dev gets feedback and fix bugs  
we don't need to test the code it's all automatic  
CD: Continous delivery: Ensures that the sw can be released reliably after the test and build. We ensure deploy  
happens often and quickly, shift away from "one release every 3 months" to "5 releases a day" 

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/7e9b3b94-9646-401f-a808-3bac71924293">

- CodePipeline: automate pipeline. Visual workflow tool to orchestrate CICD. We can control the Source / build / test /
  deploy / invoke stages. If a stage fails pipeline stops. You can create events for failed pipelines / events with
  CloudWatch events.
  you can add an "APPROVAL ACTION" to a stage in a pipeline at the point where you want the pipeline
  execution to stop so that someone with the required AWS Identity and Access Management permissions can approve or reject
  the action.
  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/bc65a9a7-7ddf-4d4c-86a5-fe9e9a28747e">  

- CodeCommit: store the code, Versioning control using a version control system. Its fully managed, SSH/HTTPS auth.
  IAM Policies, encryption at rest with AWS KMS, in transit
  - Example lambda to commit / i think other services as well to commit. The Dev can use AWS SDK to instantiate a CodeCommit client. The client can then be used with  
    put_file which adds or updates a file in a branch in an AWS CodeCommit repo.
  - connections to AWS CodeCommit repos with IAM user, configure Git credentials for CodeCommit in the IAM console  
  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/cc98d861-4d4f-4ab7-90fd-260eda8be962">  

- CodeBuild: Build and test code. The build instructions are in buildspec.yml (must be at root of code like .git).
  Logs can be used in S3 / cloudwatch. we can use CloudWatch Metrics to monitor build statistics.
  Detect failed build and triggers / alarms.
  Don't store codefuild secrets as plaintext in env. vars. Instead environment vars can reference parameters store parameters, or secrets manager secrets.   
  build logs of the failed phase in the last build attempt in the AWS CodeBuild project build history  
  CodePipeline stage can run many CodeBuild actions (not stages) in parallel.  
  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f0d2d3dc-08c8-4b10-a775-e59e55911a99">  

- CodeDeploy: automate deploy code to EC2 / lambda (help automate with traffic shift linear / canary) / ECS (only Blue-
  green). Rollback / deploy capability. Gradual deploy control (AllAtOnce, HalfAtATime,OneAtTime,Custom, Blue-green).  
  - AppSpec.yml file says how deploys should happen.  Must run a CodeDeploy Agent on the target instance.  
  - an order of the hooks for in-place deployments: ApplicationStop -> BeforeInstall -> AfterInstall -> ApplicationStart
  - defined the deployment actions in a file AppSpec.yml. Hooks allowed: BeforeAllowTraffic > AfterAllowTraffic  
  - it can pull code boundle from S3

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/99ca4948-7aae-4adb-9221-4d657205ab0f">  

- CodeStart / CodeCatalyst: Manage sw dev activities.

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/079e00e5-8737-4a92-b317-8b972e9473f7">  

- CodeArtifact: share sw packages, sw packages depend on each other to be built (code dependencies), storing and
  getting these dependencies is called artifact management We can use Resource policy to auth different packages.

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/8c76f5b0-bfe0-4f1b-8b03-3c983ecfdbe8">  

- CodeGuru: Automated code reviews (reviewer) and app performance (codeGuru profiler) with Machine Learning.

- Cloud9: IDE in cloud. can work anywhere in the world if u have internet. 

# SAM
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f2e46532-12c0-4f01-b0c9-b549809d651e">

Serverless application model basically a shortcut to cloud formation, says how the app should be deployed and should behave  
- framework for development and deploy serverless applications.  
- All config is in YAMAL code, generating complex CloudFormation from simple SAM YAML file  
- Only 2 commands to deploy to AWS (sam package and sam deploy)
- Use "sam local invoke" to run the specific Lambda function locally
- can use CodeDeploy to deploy lambda functions  
- Can help you run Lambda, API Gateway, DynamoDB

Relies on CloudFormation templates to work!! SAM templates are an extension of AWS CloudFormation templates,  
with some additional components that make them easier to work with

AWS SAM CLI lets you LOCALLY build, test, and debug serverless applications that are defined by AWS SAM templates.  
for ex. provides a Lambda-like execution environment locally

AWS toolkits: ide plugin allows you to run build test Lambda functions with AWS SAM.  
EASIER THAN CLOUDFORMATION use for exam.

The "AutoPublishAlias" property enables AWS SAM to automatically create and update a Lambda alias that points to the latest version of the function (like after a Canary deployment)  

SAM local "start-api" subcommand allows you to run your serverless application locally for quick development and testing.  
For example it creates a local HTTP server that acts as a proxy for API Gateway and invokes your Lambda functions based on the AWS SAM template.    

<img width="500" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/bde587c1-1ff8-47b3-8b14-3d5df83b8d8f">

# CDK
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/181c3088-4372-4996-9a5f-fdc502dfa23c">

Cloud development Kit:  
define cloud infrastructure using familiar language, code gets compiled into CloudFormation model (Json/Yaml)  
You can deploy infrastructure and application code together, if it doesn't compile you don't get either of them.  

Create the app from a "template provided by AWS CDK" -> Add code to the app to create resources within stacks ->  
Build the app (optional) -> Synthesize one or more stacks in the app -> Deploy stack(s) to your AWS account  

CDK Construct: library collection of constructs for every AWS resource.  
Construct hub: 3rd party and open source from community CDKs.  

The developer can test a specific Lambda function locally by running the CDK SYNTH command to synthesize the AWS CDK (generate CF template)   
Use SAM LOCAL INVOKE to run the specific Lambda function locally  
application into an AWS CloudFormation template  


# Cognito
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/17343071-a6ff-4e38-a0f3-a8a9a40080ae">

give users an identity to interact with web or mobile apps (outside of AWS)   
IMPORTANT in exam if they say "hundreds of users", "mobile users", "authenticate with SAML"

Cognito user pool (CUP):   
- are for AUTHENTICATION (identify verification). create a serverless DB of your users. sign-in function for app users.   
  Username / email / password / MFA / phone verify. / login with Google (federate identity). block users compromised.   
- when they login they get back a JSON web toke JST.   
- CUP can invoke Lambda on triggers.  
- It offers a Hosted authentication UI that you can use in your apps.  
- Offers Adaptive Authentication: block sign-ins or require MFA if the login is suspicious.  
- We can integrate it with API Gateway and ALB (offload the auth. to the load balancer) we can use 2 ways to auth:  
  With Cognito user pools, or OIDC auth.  
- you can add sign-up and sign-in to mobile and web apps and it also offers a user directory so   
  user accounts can be created directly within the user pool. Users also have the ability to reset their passwords
- You can add social identity providers (IdPs) (SPID), are based on OpenID, they reqire the "Client ID" & "client Secret"
  to work 

Cognito Identity pools (federated identities):  
are for AUTHORIZATUION (access control) AWS credentials to users to access AWS resources directly, temporary  
credentials by logging in with public providers (amazon, google, apple, Facebook), or users in an Amazon Cognito pool.  

Amazon Cognito Sync:  
creates a local cache for the identity data. Your app talks to this local cache when it reads and  
writes keys. So even when you are offline. When the synchronize method is called, changes from the service are pulled to the device, changes are available to other devices to synchronize.  

WE CAN ALSO ALLOW FOR UNAUTHENTICATEDS GUEST ACCESS With Amazon Cognito with unauthenticated access enabled.  

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/9af0fe7d-94c5-4a2b-a5fe-c132c533bf99">

# Step functions  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a419bce9-9dd6-4b93-92fd-59d51c9d550a">

perform many simple tasks, process this data through multiple business rules and transformations.  
Model your workflow as state machines (one per workflow) using Amazon States Language for order fulfillment, data processing, web app... any workflow  
Write in JSON, visualized. In task states you can invoke AWS services, like lambda, do dynamoDB stuff ecc...  
States can be: Choice (test condition) / fail or succeed / Pass (pass data with no work) / Wait / Map state /  
PARALLEL STEP - begin parallel branches of execution.  

<img width="300" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a63f3b08-3aa7-44be-b5b7-92d1dba4535b">

Error handling: we can have retry or catch (transit to failure path)  

Wait for task token: allows you to pause step function until a task token is returned. by appending .waitForTaskToken!   

Activity task:  
allows you to have the task work performed by an Activity worker (Apps on EC2 / Lambda / mobile device...)  
after it sends a SendTaskSuccess or SendTaskFailure. To set how long it should run TiemoutSettings, periodic send beat  
with SendTaskHeartBeat 

2 types of workflows:  
- Standard (default):  
  max duration 1 year  
  price by nr of states  
  used for non-idempotent actions (same request leads to the same system state, and no action is unintentionally  
  executed more than once) like payment process.
- Express:
  up to 5 min, higher capacity, billed by nr of executions, duration and memory consumption. used for IoT data ingestion,
  streaming Data, mobile app backends.
  - Async (At least once execution guarantee) Don't wait for results, ex message services you don't wait for a response.
    must manage independence to manage that if it runs twice you don't have twice the effects.
  - Sync (At most once execution guarantee) Wait for workflow to complete. When you need an immediate response. At-most
    because if there is a failure it doesn't automatically restart, it's up to your logic.  

"DefinitionSubstitution" entries allow the state machine to include resources that are declared in the AWS SAM template file or CF template  

AWS App sync:  

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/c86141c0-adb0-4b08-8784-c5ced50208ef">  

managed service that uses GraphQL: makes it easy for apps to get exactly the data they need. You ask for a field  
and that is what gets returned. You can combine data from multiple data source types, NoSQL, SQL, HTTP, APIs...  
You can also use it to get real-time data with WebSocket or MQTT on WebSocket, so if u need a field real-time  
For mobile apps: can be used for local data access and data synchronization.  
To start uploading a GraphQL query  
Security: API_KEY, AWS_IAM, OPENID_CONNECT, AWS_COGNITO_POOLS, Https with CF in front.  

AWS Amplify:  
AWS Amplify is a fully managed service that allows developers to build and deploy web applications and static websites.  
With Amplify, developers can easily connect their repositories, such as AWS CodeCommit, Bitbucket, and GitHub, to automatically build and deploy changes to the website based on code merge-  
Like EB for mobile and web apps. Gives us Data Storage, auth, ml, frontend libraries.  
gives you Authentication out of the box with Cognito.  
gives you Data Store out of the box with AppSync and DynamoDB  
amplify.yml build  
Adding a test phase to the amplify.yml build settings allows the developer to define and execute end-to-end tests as part of the build and deployment process in  
AWS Amplify Hosting.  

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a3f6a3a4-f326-4f00-a43c-4a1af6226434">

AWS AppSync:  
build scalable applications that require real-time updates on a range of data sources, including Amazon DynamoDB.  

---
# AWS Security
in flight TLS / SSL will be encrypted before sending and decrypted after. prevents MITM man-in-the-middle attack.   
Server-Side encryption: Data is encrypted after being recieved by the server, decrypted before being sent. It's stored in an encrypted for thanks to a key.  
Client-Side encryption: data in encrypt. by the client and never decrypted by server. Client-side encryption with  
customer-managed encryption keys allows you to select either symmetric or asymmetric keys.  

# KMS 
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/471d117e-2e6a-4390-a241-b0a00c50934e">

### Key Management Service
manages encrypted keys for us. Seamlessly integrated into AWS service, they need access to keys. Able to audit the Keys with CloudTrail.  
Types of keys:  
- Symmetric (AES-256 keys)
  Single key for encrypting and decryptynd
- Asymmetric (RSA, ECC key pairs)
  Public and private keys, public key is downloadable.

Types of KMS keys:  
- AWS owned keys (free) 
- AWS Managed Key: free (automatic rotation every 1 year)
- Customer-managed keys created in KMS (1$ a month) automatic rotation 1 year
- Customer-managed keys imported (1$ a month must be symmetric only manual rotation possible)

KMS Key policies: control access to keys like s3 bucket policies. Defines who has access to KMS keys. You can set which actions to allow, only generate key, encrypt,
decrypt ecc...

how encrypt / decrypt works:  

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/ad7a1c71-5da4-42ab-9e64-62d9d40a57a1">

Envelope Encryption: Encrypt plaintext data with a data key and then encrypt the data key with a top-level plaintext master key.  

Envelope Encryption: KMS has a limit of 4kb, if you want more, use Envelope Encryption which corresponds to  
GenerateDataKey API.   
GenerateDataKey: obtain an encryption key from KMS that we can then use within the function  
code to encrypt the file. This ensures that the file is encrypted BEFORE it is uploaded to Amazon S3.
GenerateDataKeyWithoutPlaintext: returns only the encrypted copy of the data key which you will use for encryption.  
returns a data key that is encrypted under a customer master key (CMK) that you specify.  

KMS limits: ThrottlingException, to respond use Exponential backoff, all services that use KMS share a quota across all regions and accounts. we can use DKE caching
or you can request quota increase through API or AWS support. 

AWS CloudHSM:  
AWS provisions encryption Hardware, dedicated hardware (hardware security module), so you manage your encryption keys entirely not AWS, supports both  
symmetric and asymmetric. Spread across multiple AZ, great for aval and durability. 

AWS SSM Patameter store:  
Secure storage for your configs and secrets, optional encryption using KMS. Security through IAM, notification with EventBridge, version tracking for changes inside.  
Parameter policies can have TTL, DOES NOT SUPPORT KEY ROTATION

AWS Secrets Manager:  
meant for storing secrets, newer than SSM, allows to force rotation of secrets every x days. Integrated with RDS, meant for RDS so in the exam if u see its Secret Manager.  
You can replicate secrets across multiple regions, and replicas sync. 

AWS Systems Manager Parameter Store:  
lower cost than Secrets Manager, doesnt have automatic rotation. If exams asks for lower cost or app handles  
rotation use this
incorporate Systems Manager Parameter Store value into the CloudFormation template: 
- if plaintext: SSM dynamic reference
- if secureString: SSM-Secure dynamic reference

AWS Nitro Enclaves:  
process sensitive data in an isolated computer environment, only authorized code can be running, fully isolated VMs. 

---
# Extra services

AWS SES:  
Simple Email service, send emails, get emails to S3, SNS, lambda  

AWS OpenSearch Service:  
In DynamoDB, query only by primary key or indexes, with open search you can search any field even partially matches. You can perform analytics with the dat  a
and use OpenSearch Dashboards to visualize it.  

Amazon Athena:  
serverless query service to analyze data stored in S3, SQL to query files. Commonly used with QuickSight for reporting / dashboard, used for business intelligence,  
analysis, report, ecc... Exam if analyzing data in s3 using serverless SQL, use Athena.   
You can improve its performance using a columnar data for cost-saving, compress data for small retrieval, partition datasets for data you check often.  
Federated query: you can query data not only on S3, but from anywhere and different types, by using an AWS Lambda.   

Amazon Managed Streaming for Apache Kafka (MSK):  
Alternative for kinesis. 

AWS Certificate Manager (ACM):  
manage and deploy SSL/TLS certificate, provide in-flight encryption for websites. Remember its stored in us-east-1 always!

AWS Private Certificate Authority (CA):  
Managed service allows you to create private Certificate Authorities, only work in your Organization / services that have the integration, not on the public internet.   
For custom Encrypted communication, authentication users, public key infrastructure.   

AWS Macie:  
Managed Machine learning pattern matching to discover and protect your sensitive data in AWS. Help identify and alert about PII  

AWS APPConfig:  
If you want your config outside of your app instead of shipping it with it. Ex. feature flags, to dynamically disable or enable features on an app. or ip blacklist ecc...  
without changing the application code.  




































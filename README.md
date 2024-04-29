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

# uncategorized
AWS limits (quotas)
- API rate limits
- Service quotas (service limits)
Exponential backoff is used when you get a ThrottlingException, so on each retry double the seconds you wait
you must implement the retries on 5xx server errors
Dont implement retry on 4xx client errors


# IAM
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/3d22a197-c740-4bcb-bc14-38ea11d542e7">

### (identity and Access Managment)
Its a global service (doesnt matter in which region i create the user, available everywhere)

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
  - "Effect": allow/deny access to certain APIs
  - "Principal": account/user/role its applied to
  - "Action": API calls that will be allowed / denied based on the effect
  - "Resource": list of res. to which the action is applied
  - "Condition": when its applied (optional)

Defense mechanisms:
- password policy (length/special chars/numbers/pass expiration, prevent reuse)
- MFA

To access AWS:
- AWS managment console (pw + MFA)
- CLI (access keys)
  - to use CLI with MFA you need to create a temporary session, you do so with the STS GetSessionToken API
  - the CLI looks for credentials in this order: Command line, Env. Variab. CLI credential file, CLI config file, Container cred. Instance profile cred.
- SDK (access keys)
  - looks for credential in this order: Java system propreties, Env. variab. The defaul credential profile file. ECS container. instance profile credentials.
Access keys are generated with the AWS console, users manage their keys and are secret like passwords.

IAM Roles:
some AWS services will need to perform actions on your behalf, ex: you give permissions to your EC2 instance to do stuff on your AWS.
to do so you need to assign a ROLE to EC2

IAM security tools:
- IAM Credentials report (list of account user status and credentials)
- IAM Accesso advisor (shows the permissions granted to a user and when were last accessed)

IAM Best practices: 
- dont use the root account exept setup
- one phisical user per AWS user
- Assign user to grous and give permissions to groups not each user
- stong password policy
- MFA
- create and use Roles for givin perms. to AWS services
- Use access keys for CLI and SDK
- audit permissions w/ Credential report and IAM access advisor

Shared reponsability model:
You know.


# EC2 Fundamentals
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/70df717a-6b4b-4bcd-b73c-5371b49c5822">

### Elastic Compute Cloud
Instance types:
- General purpuse (diverse workloads, websites or code repos)
- Compute optimized (high level of processign, machine learning, gaming server ecc..)
- Memory optimized (fast work for large data in memory, in memory dbs, cache store, real time process of data)
- Accelerated Computing 
- Storage Optimized (when lots of datasets on local storage, data warehouse, oneline transaction)
- HPC optimized

m5.2xlarge: m = class like general purpuse, 5 = generation (aws improve over hw over time) 2xlarge = size in instance class (the more cup and memory ecc..)

Security groups:
firewall around EC2, control how traffic is allowed into EC2, they only contain ALLOW rules. 
they control:
- access to ports
- autorize IP ranges
- control of inbound network
- control outbound network

Ports to know
- 22 SSH secure shell (linux instance access)
- 21 FTP file transfer protocol
- 80 HTTP unsecured websites
- 443 secured website
- 3389 RDP remote desktop protocol (windows instance access)

Connect with: SSH, putty, EC2 Instance connect

purchasing options:
- on-demand
  - pay for what you use
  - no upfront comittment
  - raccomanded for short workload inunterrupted you dont know how the app. will behave
    
- reserved (long workloads 1-3 years)
  - discount compared to on-demand
  - you reserce an instance attribute (type, region, os)
  - pay pfront, partial, no
  - raccomanded for steady-state usage (like a db)
  - you can buy / sell it on the marketplace
    
- conv. reserved instances
  - change type but less discount
    
- savings plan (1-3 years you commit not to type but to amount of workload in dollars)
  - commit to a usage, beyond that its billed on-demand
    
- spot instances
  - most aggressive discount
  - you can lose it at any time, use it for workloads resilient to failure like batch, data anal, image process, distributed workl, flexible start and stop.
  - not suited for critical jobs
    
- dedicated instance
  - when compliance req. or bound sw licenses, or strong regularitory needs
  - on demand or reserved
  - share host with others

- dedicated host
  - access to your own server and visibility on lower level

- capacity reservation
  - reserve on demand instance capacity for any duration, you have access to ec2 capacity when you need it
  - no time commitment

EC2 Instance metadata IMDS
allows aws EC2 instances to learn about themselves without using an aws role for that
http..../latest/meta-data
you can retrieve info about the EC2 but not the IAM policy
 
# EC2 Storage
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/e04b234a-8821-4d49-94e8-9295df6a2c08">

EBS (elastic block store) network drive you can attach to your drive while they run:
  - persistant data, only mounted to 1 ec2 at the time
  - locked to only 1 AZ at the time
  - network drive so a bit latency to communicate (not phisycal drive)
  - because its network it can be detached and reattached somewhere else
  - can have a delete on termination attribute

EBS Volume types:
  - gp2/gp3 "general purp." balance price and perf for wide variety workloads
    - can be used for boot volume
    - cosr effective low latency
    - virtual desktops and test environments
    - gp3 can individually increase IOPS and throughput, for gp2 they are linked together
      
  - io 1 / io 2 "prevision iops": highest performance ssd
    - can be used as boot volume
    - for mission critical low latency or high throughput workloads
    - good for DB workloads (sensitive to storage and performance)
      Multi attach family
      - attach to multiple EC2 in same AZ (only avable to io 1 / io 2)
      - for high app. availability
      - manage cuncurrent write operations
      - MAX 16 EC2 at the time
        
  - st 1 low cost hdd for frequent access throughput intensive
    - cant be a boot volume
    - throughput optimized (big data, data warehouse)
      
  - sc 1 lowst cost hdd for less frequent access
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
  - better I/O performance, high performance because phisically attached hw store
  - lose the storage if ec2 is stopped
  - good for buffer / cache / scrath data / temp content
  - risk of data if datacenter hw fails
  - your respinsability to backup

# EFS
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/02c35483-16c1-4347-833b-cafe9c6dc073">

Elastic file system (network file system can be mounted on many EC2)
  - they work with EC2 in multiple AZs
  - high avalability, pay per use
  - security gropu to control access to EFS
  - compatible with AMI
  - encryption at rest with KMS
  - Scale automatically and is pay for use
  EFS Performance classes:
    - EFS scale
    - performance mode
    - throughput mode
  EFS Storage classes:
    - standard: frequent access
    - EFS-IA infrequent access, lifecycle policy says after how much it gets converted
    - EFS One Zone-IA best cost saving
    

# AMI
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c8c0f021-4aef-4778-827b-697415ef7520">

### amazon machine image
it powers the EC2 instance, rapresent a customization of EC2 instance
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

### elastic load balancer
its a MANAGED load balancer, aws guarantees it will be working and will upgrade it automatically
what it does:
- forwards traffic to multiple servers
- expose a single point of access (DNS) to your app
- handle failures of instances, using health checks, by using a port and route to check the http response.
  ex response 400 not healthy


4 types:
- classic load balancer: deprecated

- application load balancer: routs https, websocket traffic
  -  load bal. to multiple http apps. across machines (target group), ec2 instances/ec2 tasks/lambds
  -  load bal. to multiple apps on the same machine (containers)
  -  supports route routing / query strings like example.com/users
  -  we can do stuff like if the request is ?=mobile it redirects to a specific instance otherwise if ?=desktop to
     different one

- network load balancer: forwards tcp udp lts traffic
  - works like app. load balancer, we redirect to certain target groups
  - redirects to target EC2 instances
  - redirects to ip addresses
  - supports tcp,http,https health checks
    
- gateway load balancer: operates at layer 3 network layer, scale and manage 3rd party virtual applications
  for example firewalls, intrusion detection system, inspection system... for example if u want your traffic to be
  inspected before it reaches ec2
  - GENEVE protocol 6081
  - target adresses: EC2 / IP adresses

Sticky sessions: its possible to implement so that the same client is always redirected to the same instance behind a 
load balancer, it uses a cookie with an expiration date. it can be an Application cookie or duration-based cookie.

Cross zone load balancing: each load balancer distributes evenly across all instances in all AZs 
or without all ASz recieve the same amount no matter how many instances in the AZ.

can be public or private

SSL/TLS certificate allows traffic between your client and load balancer to be encrypted in transit, they are
issude by Certificate Autorities, they have an expiration date

Connection Draining: If EC2 is shutting down it goes in draining mode so the uses that are already connected and need
to complete their requests finish and ELB doesnt forward new traffic there, then when its all done it shuts down.

load balancer security group allows traffic from anywhere, in routing table 0.0.0.0/0
EC2 security group allow traffic only from the load balancer, so in routing table the source is the load balancer name

# Auto scaling group
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/f861c0d3-3b2a-4546-a6fd-2ab0a6252823">

in cloud u can get rid or add servers quickly
the goal of ASG is to 
- scale out (add EC2) to match increase in load
- scale in (rem. EC2) to match decrease in load
- ensure we have a min and max number of EC2 running
- registed new instances to ELB
- ASG launch template contains info on how the new instance should be created
- its possible to scale based on CloudWatch alarms metrics
- after a scaling happens there is a default 300 sec cooldown period

Scaling policies:
- dynamic scaling: based on tracking scaling ex: targeto to have ASG CPU to stay at 40%
- simple / step scaling: when a CloudWatch alarm is triggered
- scheduled scaling: anticipate scaling based on pattern
- predictive scaling: pattern that repeat based on forecast

Instance refresh:
if you want to update the launch template, as scaling happens old instances get terminated and new ones
will have the new template, so at the end after a while all instances will have the new template

# RDS
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/b70b6dcf-bc97-462c-8db3-30509fcb3ed4">

### relational database service
Managed DB service, uses SQL, because its managed, rather than EC2 own DB:
- OS patching by AWS
- continious backups and restore
- read replicas: help you scale your reads (not writes)
  - async replications so the reads are "eventually consistant" because if they havent finishes syncing you can read
    old data sometimes
  - up to 15 read replicas
  - cross AZ or cross region
  - can be converted to their own DB
- multi AZ setup
  - failover if disaster recovery
  - replicas can be used for mutli AZ disaster recovery
  - one DNS name, automatic failover
  - sync replication 
- scaling capability / Auto scaling
- storage backed by EBS

RDS proxy
fully managed db proxy for RDS
- allows apps to pool and share DB connection, minimizing timeouts, open connections, and reduce stress on DB resources
- auto scaling, serverless, across multiple AZ
- apps connect to proxy not directly to db instance

# Aurora
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c84aa5c9-1953-48d1-8304-e702b1b3c738">

priorietary tech from AWS
- SQL DB, 5x & 3x times faster
- The storage automatically grows
- up to 15 replicas
- automatic native failover
- automatic backup and recovery
- 6 copies of data in 3 AZ
- 1 of the 6 replicas in a master that writes
- 5 of the 6 are read, you use a Read Endpoint that connects balances the request to the and decides from which one to read
- automatic patching /maintance

both RDS and Aurora:
- at-rest encryption: with AWS KMS
- at-flight encryption wusint AWSTLS
- IAM authentication: IAM roles to connect / pw username
- security grous for network access
- audit logs can be enabled

Elastic cache:
redis / memcached
in memory database with high performance low latency
- common query will be cached
- its managed, so aws takes care of the usual stuff
- Application queries ElastiCache, if avable gets from there, otherwise query RDS and store in ElastiCache
(cache hit or cache hit)
- or you can cache the session data for faster
- cached data may be eventually consistant
- eficcient if data changes slowly
- Lazy loading / Cache-Aside / Lazy Population architecture: cache hit / cache miss depending if the data is in ElastiCache, if not get from RDS and write it to cache for further searches.
- write through architecture: write to cache when there is a modification to the DB


Cache eviction:
- you delete item explicitly
- evicted cuz memory is full and not recently used
TTL time to live:
- used for leaderboards / comments / activity streams
- can range from a few seconds to days

# Route 53
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/72e0e111-ab7b-44fc-a095-c86f2704330b">

DNS domain name system: translates ip addresses to human friendly names
terminology:
- Domain registrar: Route 53, godaddy..
- DNS records:
- zone file: contain DNS records
- name server: resolver DNS queries
- top level TLD, second level SLD  
<img width="450" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/d48cb9cf-8a74-4eb9-a5ba-182dd28d3a23">

Route 53 is an authoritative DNS (you can update the records youself), and is a Domain registrer
Route 53 records contains:
- Domain/subdomain name: example.com
- record type: (must know)
  - A maps hostname to ipv4
  - AAAA maps hostname to ipv6
  - CNAME Point a hostname to another hostname, works if you have a non root domain (something.domain.com)
  - Alias: Point a hostname to AWS resource, works with root name (ELB, CloudFront, API Gateway, s3, VPC)
  - NS name server for the hosted zone controls how traffic is routed
    -  public hosted zones responds with the public ip when trying to resolve the DNS
    -  private hosted only you in your VPC network can access it
- value: 128.241.223.111
- routing policy: how r53 responds to queries
  - Simple, route traffit to a single resource, if multiple values (ip adresses) are returned che client chooses a random one
  - Weighted: control the % of the request that go to each resource
  - Failover: done with health checks, if one is not healthy, switch
  - Latency based: redirect to the resourch that has the least latency
  - Geolocation: based on where the user is olocated
  - Multi-Value: used when routing to multiple resources
  - ip-based: routing is baded on clients IP address
  - Geoproximity: based on a value called bias, which is the radius  
- TTL amount of time the rcord is cached at DNS resolver

traffic flow: node based editor allows to config complex routing decition policy 

Health checks are for public resources
- this allows automated dns failover

Route 53 Records TTL:
- the client will respond with the cached ip if TTL hasnt expired yet.

domain registrar: allows you to buy your domain value (godaddy) and they give you usually a way to manage your domain record
but you can use a different one if needed

# VPC
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/99820e8a-bbf2-41f7-aca8-44d30ce6a14e">


### virtual private network
subnet allow to partition network inside VPC
subnet can be public / private
to define access between subnets and how it flows between sections use Routing Tables

Internet gateway: help VPC instances connect to internet

Public subnet has a route to the intert gateway

NAT gateway (aws managed) and NAT instance (self managed) allow your instances in private subnet to access the internet while remaining private

network ACl (NACL) firewall which control traffic from and to subnet
can only allow or deny rules
is stateless return traffic must be explicitly allowed

security group: only the allow rule
firewall tha tcontrolla trafico from EC2 instances 
stateful return traffic is automatically allowed

VPC flow logs
capture info about IP traffic 

VPC peering: connect 2 VPCs privately
must not have overlapping CIDR

VPC endpoints
allow to connect to AWS services using a private network instead of public network

connect on premise site with VPC
VPN encrypted conn over public connection
Direct connection: phisical connection to AWS

3 tier solution architecture
Route 53 -> ELB public subnet (1) -> private subnet (2) with 3 EC2 -> data subnet (3) RDS / elastiCache

LAMP stack on EC2
Linux
Apache
MySQL
Php

wordpress on aws 

# S3
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/c0ddee13-1fc0-4509-a333-766fd99bd775">

Used for everything, is the backbone for a lot of internet stuff
backup and storage, disaster recovery, archive, hybrid cloud storage, app hosting, media hosting, data lakes and big data, sw delivery, static website

store objects (files) in "buckets"
must have unique name
defined at the region level ( not global level)

contains:
- each object has a key: the full path of the file, prefix + object name
- max size is 5TB, files > 5gb are multi part upload
- metadata list of key-val per object
  - name must begin with "x-amz-meta-"
  - retrieved when retrieving the object
  - cant filter by metadata
- tags key-val for security / lifcycle
  - useful for fine-grained permission
  - cant filter by tag
  - good for analytical purpuses
- version ID (if versioning enabled) 

security:
- IAM policy for users
- Bucket policy; bucket wide rules at the resource level
- object ACL
- bucket ACS
- encrypt the object server side SSE 
  - SSE-S3 server side encryption with s3-managed-keys
    - key handled and managed by AWS
    - AES-256
    - enabled by default
  - SSE-KMS server side encryption with KMS keys stored in aws kms
    - user control over keys, can audit who uses it with cloudtrail
  - SSE-C server side encryption with customer-provided keys
    - AWS does not store the key you provide
    - must use HTTPS
- encrypt the objects client-side
  - encryption and decryption outside of AWS
- encryption in transit
  - SSL/TLS encryption while its being transmitted, using the HTTPS endpoint
    
can force encryption with bucket policy

- CORS: cross origin resource sharing
  its a security in web browsers to allow request from other website origins
  (popular question) if a client makes a cross-origin request on our S3 bucket we need to enable the correct CORS headers

- MFA delete
  require MFA when deleting an object

- S3 access logs
  you can enable logging of all accesses and operations. The target of logs should be a different S3 bucket never the same one

- pre signes URLS give access to a file on a private bucket from 1min to 12hrs, using a presigned url by you

- S3 access points:
  Users from finance can only access /finance/... files, sales can only access /sales/... files
  this is done with an access point policy

- S3 Object lambda
  if u want to run a function before its being retrieved, use and object lambda access point.
  ex: adding watermark specific for the user, resizing, converting the data

S3 can host static websites and have the accessible public with a policy

you can version your files, its enabled at the bucket level, instead of overwiting it creates a new version
you can enable s3 versioning to have objects that when are deleted are hidden and can be recovered

S3 can generate pre-signed urls to issue a request as the person who signed it, using the IAM key of the signing principla, it has a limited lifetime

replication:
enable versioning 
- CRR cross region replication (compliance or low latency)
- SRR same region replication (live replication, log aggregation)
- buckets can be in diff AWS accounts
- copy is async

Storage classes:
- general purpuse
  - frequently accessed data
  - low latency high throughput
  - for big data analytics, mobile gaming, contenti distribuition
- Standard-IA
  - For data that is less frequently accessed, but requires rapid access when needed
  - for disaster recovery / backups
- One Zone-IA
  - in a single AZ
  - secondary copy of backup or data u can recreate
- Glacier Instant Retrieval
  - millisecond retrival
- Glacier flexible retrieval
  - 1-5 min
- glacier deep archive
  - 12 hrs
- intelligent tiering
  - move objects between tiers based on usage

durability: how many times an object is going to be lost 9.99 11 9s  
avalibility: how readly a service is: 99.99% not aval for 53min

can use lifecycle configurations or move between classes manually
lifecycles rules:
- Transition Actions: configure objects to transition to another storage class after a certain time
- Expirtation actions: configure objects to expire (delete) after a certain time / delete old files / del partial files

S3 Event notifications:
like generate thumbnails of images uploaded to s3
S3 bucket -> Amazon EVent Bridge -> rules -> services

S3 performance
- can speed up upload by using multi-part upload
- s3 transfer acceleration
- byte-range fetch: request a specific range of bytes in a file to speed up downloads (only partial data)
- sql server-side filtering, the server sends data already filtered

# CloudFront CND
<img width="50" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/eee7115d-40b2-432c-8b06-0d267cd3e9cc">

Its a CDN content delivery network
impoves read performance, content is cached at the edge
DDoS protectionintegrates with shield
It integrates with ANY HTTP backend you want, S3, app load balancer, EC2, S3 website
CLient asks to Edge location (cache), if it doesnt have it the edge location gets if from the origin

CloudFront vs something like S3 replication
CloudFront:
- is global edge network
- files are cached to each edge location TTL for a day
- great for static content that must be aval eveywhere
S3 replication
- must be setup for each region you want replication
- files are updatet in near real time
- great for dynamic conent that need to be aval at low latency in few regions

The cache lives at each cloudFront edge location
you want to maximize the cache hit ration
object are identified using the Cache Key
- unique id for every object
- consist of hostname + resource portion of URL
- if you want info not static, like based on user device / language /location add other elements to HTTP headers using CLOUDFRONT CACHE POLICIES
  - define cache based on HTTP headers, cookies, query strings
  - controls the TTL
  - All info you include here is automatically included in the origin requess
  - YOu can use an Origin Request policy for vals that you want to include in origin request wihtout including them in cache key  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/26b2a180-f877-462e-bba7-f443c36d60cc">

Cache invaludation:
If you update your backend origin, CloudFront doesnt know about it and will only change after the TTL
You can force and entire or partial cache refresh bypassing the TTL using cloudfront invalidation

you can config different cache settings for different content type or path pattern (ex maximize hits by separating dynamic and static content)

ALB or EC2 as HTTP background
instances must be public there is no private VPC connection from edge location
so the resources must allow traffic from public ip of edge location
If it's an ELB it must be public, if its EC2 it can be private because there is VPC connection from ELB to EC2

Geo restriction
allowlist: if you are on the list of approved countries
blocklist prevetnt user fron entering from banned countries

Signed URL / signed cookie
Signed URL = access to individual files ( one url per file )
Signed Cookies = access to multiple files at once
for example premium private users who paid for shared content
we can use a URL / Cookie we attach to the policy that includes:
- url expiration
- ip range
- who is the trusted signer

Cloudfront signed URL vs S3 pre signed url
presigned you have access to directly the S3 without being able to use cloudfront, it issues a request simulating the person who pre-signed the url

Signed URL process:
2 types of signers, either a trusted key group ( raccomanded ) or an aws account that contain a key pair (not raccomanded)
create one of more trusted key groups, the public key is used in the URL, the private key is used by your app.

Pricing:
the cost of data out per edge location varies
you can reduce the number of edge location for cost reduction usign the 3 cloudfront price classes

multiple origin:
route to different kind of origins based on the content type or path
for example /api/* requests from ALB, /* requests from S3 bucket

origin groups:
to increase high avalability and do failover
one primary and one secondary origin, if the primary fail the secondary is used

Filed level encryption:
protect user sensitive info through application stack
sensitive info encrypted at the edge close to user

Real time logs:
get requests recieved by cloudfront and sent to Kinesis data streams, here you can monitor, analize and take action based on content performance

# Docker
- software development platform to deplay app
- apps are packaged int containers that can be run my any OS, they run the same regardless of where they run
Docker VS VMs
resources are shared with the host, many containers on one server, they can be managed by ECS, EKS, ECR, Fargate  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/25b17b94-3cab-4d38-b615-95f0e1fe9864">

# ECS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/7701684b-84d8-4979-bf61-f32db171ac1a">

### elastic container service

EC2 Launch type:
to launche a container = launche an ECS task
If you decide to use an "EC2 launch type" (EC2 inside ECS and ECS agent with docker on each EC2) You must provision and maintain the EC2 instances that run inside ECS
- the EC2 instance makes API calls to ECS services
- conds container logs to CloudWatch
- pulls Docker images from ECR
- individual taks can have roles to API calls with AWS services
- Load balancing: Dynamic host port mapping, each task has a diff random port,
  the ELB finds the correct ports automatically even if random. You must allow on the EC2 instance security group any port from the ALB security group
- for EC2 instance storage to share data between tasks
- ECS Task Placement:
  when a task of EC2 is launched, ECS must determinate where in which EC2 to place it for memory and CPU avalability
  to assist you, you can define a Task placemenet strategy and task placement constraints
  - Binpack: Place tasks based on the LEAST available amount of CPU or memory, cost saving so it usese the least possible amount of EC2s
  - Random: places a task randomly, no logic to it
  - Spread: Places tasks evenly based on a specified value: ex spread evenly along AZs
- ECS task placement contraints:
  - distinct instance: place each task on a different container instance, so never 2 task on the same container
  - memberOf: place a task on the instance that satisfy an expression

Another type of "launch type is *Fargate*", you dont need to provision the infrastructure, serverless, 
just run EC2 tasks, automatic scaling, is fully managed
- load balancing: each task has a unique private ip, you only define the container port. the ALB will
  connect all incoming connections to the same port.
- fargate ephemeral storage tied to lifecycle, shared between containers
- ECS task placement ins automatically handled

Load balancer integration
we can run an ALB for different instances with tasks

Data volumes
we can use EFS file system onto ECS to have a file system for all tasks, tasks running in any AZ will share the same data in the EFS. 
Fargate + EFS = serverless, you cant mount s3 as a file system here

ECS service auto scaling
automatically increse / decrease tasks
we can scale on CPU / Memory / ALB requests
ECS services to scale undelying EC2 instances:
- ECS Cluster Capacity Provider is what is used to automatically provision and scale the infrastructure,
  adds EC2 when you are missing capacity (CPU, RAM) (raccomanded)
- Auto scalinng group scaling, scale based on CPU untilization
we can scale using
- target tracking based on a target value from cloudwatch metric
- step scaling scale based on a specified cloudwatch alarm
- scheduled scaling scale based ona specified date / timeW

ECS Rolling updates
when upgrading from V1 to V2 we can control how many tasks can be started and stopped and in which order in order 
to roll updates to different tasks by starting the new ones with v2

ECS solution architectures
serverless object pocessing example with ECS
user uploads image to s3 bucket
even bridge runs ECS2 task
the task now gets the object from the bucket
it saves the results in Dynamo DB  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/46bfc7f7-1b41-4450-818b-6a312fb2323a">

another serverless example using scheduled task every 1 hr  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/0ec7cc33-ad03-47ce-92d6-0da32133f6c8">

tasks pool from SQS and if more messages, use auto scaling to handle them  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/df9fc722-bac7-4d56-b049-bdfec728587e">

Event bridge allows you to handle lifecycle of your tasks  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/d2c60519-8734-4fb9-af5f-87043fc47de8">

ECS task definition
metadata JSON tell how to run Docker containers like AMI for ECS
- port binding
- cpu / memory
- environment variables: hard coded like URLs, SSM parameters (eg API keys, shared configs). You fetch either from SSM paramater store or Secrets manager
- IAM roles: task definition, each tasks in a service get this role, you can define different roles per services
- logging config
- network info
- Data volumes: share data between containers in the same task definition, so like some task can write to a path, and a logging task can read from it and save them.
  for EC2 instance stoagre, for fargate ephemeral storage tied to lifecycle.

Amazon ECR
elastic container registry, used to store and manage docker images on AWS
private repo for your account or public repo using ECR public gallery
IAM role to EC2 instance to pull ECR repository

AWS Copilot:
CLI tool to build release and operatre containerized apps.
it provisions all required infrastructure for containerized apps (ECS, VPC, ELB, ECR...). Auto deploy with code pipeline + deploy to multiple environments

Amazon EKS
another way to run containerized apps. using Kubernetes not ECS, runs docker containers but is open source and used by more different cloud providers
same goal but different API. supports EC2 mode or FARGATE mode. 
Its Cloud agnostic so it can be used in any cloud, use it if your company / another cloud you use, uses it. 
You create EKS nodes with tasks inside, they can be: Managed node groups (auto scaling, on demand ec..) by AWS or Self-Managed Nodes.
EKS data volume supports EBS, EFS, FSx, by specifying a StorageClass. It uses a Container Storage Interface CSI, to be able to use them

# Beanstalk
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/2902359b-4e68-430a-b99a-a9da6e41c116">

Allows you to skip the manual setup for each service when deploying an architecture.
Its a managed service that uses components seen before (EC2, ASG, ELB, RDS...) but you dont manually connect them and config them, you just focus on the code development
we still have full control over the config if needed. The service is free but you pay for the underlying resources it deploys.
Components:
- Application: is the collection of all components
- App. Version: version of your app. code
- Environment:
  - contains the collection of aws resources running
  - tiers: services tiers
  - you can create multiple environments

Example of architectures:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4dc3c7dd-353e-4781-8466-769581ccbef4">

deployment mode:
single instance: great for dev
High avalibility with load balancer: great for production

Deploy options for updates:
- All at once, deploy the new v all in one go but it has a bit of downtime  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/28d406a4-c6d7-4b02-84cb-74e1349db53b">
  
- Rolling: update a few instances at a time (at one point it funs at a smaller capacity  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a16b4ed3-21aa-48a3-b73b-96fe01d23ddb">
  
- Rolling with additional batches: like rolling but actively starts new instances to swtich, so it has all the capacity when needed  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/2cb0caf7-d0db-43b9-ad76-f62eeb576408">
  
- immutable: deploys all to new instances the switches everything at once when its ready  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/cc51490f-db72-457f-8968-4b756dcd4f67">
  
- blue green: create a new environment and test, and switch when ready  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/051954d9-e3eb-4e27-8f85-60f0c180447b">
  
- traffic splitting: sends a small % of traffic to new deployment  
  <img width="250" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/07331a37-8b38-4f7f-850e-97060e09a667">

Elastic Breanstalk CLI, helpful to automate pipelines

EB Lifecycle policy: policy to remove old verions

EB extentions: a file in the .ebextentions/ directory in our code that can configure all the paramaters you find in the web UI
Because it uses CloudFormation under the hood, the extentions provisions any service you want.

EB cloning: clones an environ. with the exact config. useful for "test" versions of app. After cloning u can change settings.

EB migration: 
for example ELB cant be changed (only configured) after cloning, so we need to migrate:
create a new env. with the same config except LB (cant clone)
deploy the app onto new env.perform a swap.

# CloudFormation
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/1fe959c9-96f0-4dd9-a07c-82fcd3c052e7">

Declarative way to outline your aws infrastructure for any resource
infrastructure as code
can create a template in a Manual way: edit them in cloud formation designer or code editor
automated way: using AWS CLI 

YAMS and JSON are languages you can write CF templates

template components:
- template version
- description
- resources: are the core of your CloudFormation template, rapresent the components that will be created, res. can refer each other
- parameters: they are a way to provide inputs to your AWS CF template (ex choose type of ec2, password...)
- mapping: fixed vars within CF template ex (dev vs prod)
- outputs: output the vars such as VPC ID and subnet ID to network stack
- conditions: control the creation of res. based on conditions ex: what region u deploy in
- Instrinct functions (the most importants):
  - Ref: returns the instance ID
  - GetAtt: returns the value of a specific attribute, ex: when creating and EBS vol you want to get the AZ of the EC2
  - FindInMap: returns a named value from a specific key
  - importValue: import values that are exported in other stacks
  - Condition Functions: if / and / not / or
  - Base64: convert value to base64

CF Rollbacks
if a stack creation fails you have the option to defailt: everything rolls back, or to disable rollback to troubleshoot

CF Service Role
IAM role that allows CloudFormation to CRUD resources, gives users the ability to CRUD even if they dont have permissions to work with the resources in the stack

CF Capabilities
CAPABILITY_IAM to enable CF to create or update IAMs
CAPABILITY_AUTO_EXPAND enable if you need stacks within stacks for dynamic transforms of the CF
insufficientCapabilitesException: exp. thrown if the capab. havent been aknowledged when deploying a template.

CF Deletion policy:
Control what happens when template is deleted or when a res. is removed from CloudFormation template
- Default DeletePolicy Delete
- DeletePolicy retain = specify res. to preseve in case of CF deletions
- DeletePolicy Snapshot = take a last snapshot before del.

CF Stack policies:
JSON doc that defines the update actions that are allowed on specific res. during stack update.

CF Custom resources:
define res. not yet supported, custom res from 3rd party. They are integrations backed by lambda functions

CF stack sets:
CRUD stacks across multiple accounts and regions with a single opeartion

# SQS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/986a4ac4-0e84-4b1a-8731-f06774f3b7ae">

### Simple queueing service
Producer sends message to SQS, like with SDK (sendMessage API)
the message remains until it gets consumed and deleted (def. 4 days max 14 days)
it can have duplicated messages

A Consumer polls messages and process it and deletes it from queue
Fully managed service used for decoupling apps. max 256kb message. running on EC2 or lambda or your local server
max 10 messages at a time. after consuming delete the message so no other consumer uses it.
Can have multiple consumer at a time. we can horizontal scale by adding more consumers. Atleast once delivery because queue can have duplicate messages

We can enable Auto Scaling, the queue has an ApproximateNumberOfMessages which can be increased by setting a cloudwatch alarm that increases it

sqs has unlimited nr of messages and unlimited throughput, so we can scale fronted and backend indimendently and decouple them  
<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/59d0b910-d7e0-4381-9a51-42c20d4fcf22">

SQS securty:
inflight encryption with HTTPS API
- at rest encryption with KMS keys
- client-side if user handles it himself
- AMI policies to regulate access
- SQS access policies (if u want corss account access or publish S3 event notifications)

SQS Message Visibility TImeout
After a messaged is polled by a consumer it becomes invisible to other consumers, it has 30s to process it,
if the message hasnt been deleted cuz not processed yet, it will be put back in the queue and be processed twice, if u need more than 30s to process it
use the "ChangeMessageVisibility" API to change the timeout time

SQS Dead Letter queues (DLQ)
We can set a treshold to how many times a message can go back into the queue because a consumer fails to process it, after the "MaximumReceives" treshold
is exceeded it goes in the deal letter queue. DLQ useful for debugging, messages in there expire after the retention period, so set a higher 14days for the DLQ
Debug the DLQ, fix your code, use the "Redrive to Source" to put the DLQ messages back in the Source queue so it can be processed correctly this time

SQS Delay queue
Delay a messages so cosumers dont see it immedialty, up to 15 min, or per message individually, default is 0

SQS Long polling
When a consumer requests a message from the queue it can wait for messages from queue to arrive if there are none in the queue, this is called long polling
long polling decreasese the number of API calls, messages get processed at the end of the wait period, higher efficency
the wait time can be 1-20s, its bette than short polling

SQS Extended client
library that uses an s3 bucket to send messages higher than 256kb, the queue sends a message telling the consumer to get the larger message from the s3

Must know API
CreateQUeue, Delete queue
PurgeQUeue, deletes all messages
SendMEssage, RecieveMessage, DeleteMessage
MaxNumberOfMessages (for recieving) def. 1 max 10
RevieceMessageWaitTimeSeconds: long Polling
ChangeMessageVisibility: change message timeout

FIFO queue
Deduplication methods: if u send the same message twice within 5 minutes it gets discarded
- Content-body SHA256, if 2 are found it discards
- Explicit-body, if 2 messages with explicit body are found it gets discarded

FIFO queue Message grouping:
We can give the queue messages a MessageGroupID, so it gets in the same queue but gets processed by different consumers,
so we can have type A, B, C ecc... messages and each one type gets processed by different conusumers

# SNS
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/ffc2f900-e207-4a4f-b527-f5c8ac5ab1f0">

### Simple notification system
if u want to send the message to many recievers at the same time, email, sqs queue, shipping service ecc....
It uses the pub/sub method. A buying service publishes the message inside and SNS topic, and subs of the topic listen to the SNS topic notification
subs can be Lambda, email, SMS, kinesisi data stream, SQS, HTTP endpoints, and many services


SNS securty same as SQS:
inflight encryption with HTTPS API
- at rest encryption with KMS keys
- client-side if user handles it himself
- AMI policies to regulate access
- SNS access policies (if u want corss account access or publish S3 event notifications)

Fan Out pattern:
push the same message to multiple SQS by pushing it only once. SQS are a sub of SNS
ex. for S3 evenets to multiple queues.

SNS message filtering:
JSON policy to filter messages and only send the messages of a filter in a specific sqs queue.ù

# Kinesis

Easy to collect/process/analize streaming data in real-time, such as Application logs, metrics, website clickstreams, IoT

- Kinesis Data Streams: capture, process and store data streams
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5c8f258e-1e14-4a5f-95a5-685f309db280">

  is a way to stream big data in your systems (ingest data at scale), is made of multiple "Shards" per stream (can be used to scale up or down),
  data will be split to Client. it sends a record to the stream that is made out of Partition Key and Data Blob (max 1MB)
  after it reaches KDS it gets send a consumer, with the Partition Key, Data Blob, and Sequence Nr.
  Partition key hashed a uinique ID you pass, and it always send the data from that provider to the same shard. Multiple hashes from multiple
  sources can be sent to the same stream, so its distributed. 
  - Retention of messages 1-365 days
  - because of this: ability to reprocess data
  - once data is inserted in kynesis it cant be deleted (mmultability)
  - Capacity modes: prevision mode: fixed nr of shards, scale maually with API, you pay per shard provisioned per hour
  - Capacity modes: on-demand mode no need per provision or manage the capacity, scale based on throughput during the last 30 days, pay per stream per hour
  - Security: encryption at rest, in flight with HTTPS, IAM policies, VPC endpoints
  Producers: SDK, Kinesis Producer Library KPL, Kinesis Agent, monitor log files.
  ProvisionedThroughputExceeded: too many inputs in a shard, solution: use highly distributed partition key, retries with exponential backoff, increase shards

  Consumers: custom with SDK, Kinsesis Client Library, Lambda ecc... can have multiple consumers getting from the same shards.
  Shared fan out consumer: max 2mb/s split between all consumers of a shard (pool model)
  Enhanced fan out Consumer: 2mb/s per consumer per shard by subscribing (push model to subscribers)

  Kinesis Client Library KLC
  Java library that helps read records from Kinesis Data Stream with distributed apps. sharing the read workload.
  Each shard is to be read by only one KCL instance, 4 shards = max 4KCL instances, 8 = 8 instances

  Kinesis operations:
  - shard splitting: increase the stream capacity: used to divide "Hot shards"
  - Merging shards: decrease capacity and save costs by merging them. both used to scale up and down kinesis

- Kinesis Data Firehouse:
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/41492d0c-faaa-4e15-8d66-c1bd21b45c8c">

  Managed (ingest data at scale) used to load data into specific services, streams into ASW data stores near real time, fully managed serverless.
  Takes data form producers / kinesis data stream. And Batch write data into destinations to:
  (to know)(S3, Redshift, Amazon Open Search, 3rd party destinations, HTTP endpoints) custom destinations
  it can first transform the data by compressing/converting ecc.. or custom with Lambda
  it has auto scaling not like KDS, there is no storage so no replay capability.
  
- Kinesis Data Analytics:
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/fb8c6601-9e33-4380-b85e-e160cb48ef3d">

  analyze data Streams for SQL, fully managed.
  It can read either from KDS or KDF, and apply SQL statements to perform real type analytics + can add s3 to join data
  it can then send the data to KDS (that can do real time processing of the data) or KDF ( that can send it to S3, Redshift, other)

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
  for every service in AWS, its a variable to minotr ex CPU, networkIn, they have timestamps and you can create dashboards with them
  EC2 instance metrics every 5 min, there is a "detailed monitoring" for a cost you get every 1 minute. By default no log from EC2,
  need to setup an "agent" to push them. "CloudWatch Unified Agent" allows you to get more granular extra data form EC2 instead of the default ones
  You can define Custom Metrics, using the API PutMetricData
- Logs:
  - log groups: name rapresenting the application
  - log stream, instance withing application
  - log expiration policy
  - can send logs (export) to multiple sources. Either using S3 Export, export in bactch when running the API CreateExportTastk Or
    Using "CloudWatch log subscription" realtime logs 
  - logs encrypted by default, you can use your own KMS
  - "CloudWatch Logs Insight" used to query the logs
  - "CloudWatch metric filter" filter expressions (ex. count "ERROR" in logs) to trigger alarms
- Alarms:
  used to trigger notifications. status OK ISUFFICIENT_DATA ALARM
  it can do 3 things
  - Stop / terminate / reboot / recover EC2
  - Trigger Auto Scaling
  - Send SNS notification (here we can attach lambda and do anything)
  we can use Cmposite alarms to monitor multiple metric at a time, useful to filter "alarm noise"

Cloudwatch Synthetics scripts that monitor APIs, URLs, Website workflow eccc and check the avalability and correct operation of these elements
reproduces what customers do on a website for example, access to Headless chrome

# EventBridge
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/c3fd578a-6ee7-4551-9f57-7404a8375f8e">

- schedule scripts / cron jobs in the cloud
- react to event pattern ex: IAm user signing in -> send email with SNS, it can also trigger lambda, step function, kinesis ecc..,
  or if CodeBuild Fails, if an EC2 is   started, if S3 uplaod ecc...
Its the default event bus for AWS services, it can react to events from 3rd party "Partner Event bus", or custom Apps "Custom Event Bus"
You can archive events or replay archived events
Resource-based Policy can deny events from other AWS acc, or aggregate from multiple acc. into one using "Multi-Accont aggregration"
  
# X-Ray
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/250aee0d-120c-4f36-ae9f-2b3b9136beb4">

It has a Visual analysis of your application
Troubleshoot app. performance and errors
You can understand dependencies between microservicess
understand where the error is
It uses tracing, each component dealing with the request adds its own "trace"
IAM, KMS
To enable it:
modify your code using x-ray SDK then install the xRay deamon or enable AWS integration
How to instrument your code: instrument = measure the products performance, siagnose erorrrs and write trace information. you use
the X-Ray SDK. 
Segments: each app. will send them.
Trace: segments collected together to form an end-to-end trace.
Sampling: decrease the amount of requests sent to x-ray. control the amount of data.

AWS Distro for OpenTelemetry
AWS-supported distro of the open-source project Open Telemetry
Collects distributes traces and metrics from your apps. Its like xray but open source.

# CloudTrail
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f112834f-04ad-4f4c-b183-937edb7bd2b7">

Internal monitoring of API calls being made
Audit changes to AWS resources by users.
History of all evenets like who deleted something, can put logs into CloudWatch Logs or S3
3 type of events:
- managment events: anything that modifies your res. or aws account (enabled by default)
- data events: S3 object-level activity ex: S3 GetObject, DeleteObject ecc... or AWS lambda ececution activity (all not enabled by default) 
- Inseights evenets: because of many logs and types, it automatically analizes your logs and detects unusual activity / patterns based on your usual baseline
event retention: stored for 90 days, if u want longer, send to S3 and use Athena to analyze them.

You can have Event Bridge integration ex. User changes IAM Role -> CloudTrail -> triggers an event -> EventBridge -> SNS email

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/30e0f21a-9804-461c-8a8d-6a1864818194">

# Lambda
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/eba57b02-7f3d-479b-adc4-f2e0feb4e258">

Serverless: just deploy code without provisioning / managing servers. 
Virtual function no server to manage
mag 15min execution
runs on-demand when it gets invoked
scaling is automated
pay just for request and compute time

Main integrations: 
API Gateway: to create rest API
Kinesis: data transofrmation on the fly
DynamoDB and S3: triggers for when something happens in DB
CloudFront: lambda edge
CloudWatch events / EventBridge: when we want to react when something happens in our structure, or if we want to use a CRON eventBridge rule.
CloudWatch Logs: stream the logs where we want
SNS: React to notification
SQS: process queue messages
Cognito: react for example to logins  
ALB: HTTP client call to ALB -> ALB convert HTTP to JSON request and invokes Lambda synchronusly  
<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/858dd518-f375-44a1-bd80-0a1c2abc0613">

<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5976dd42-d6d8-4e95-aa0b-433917a1184b">

Ways to process data:
- Synchronus invocation: happens when using CLI, SDK, API Gateway, ALB, Cognito, step functions, cloudfront (all user invoked or service invoked services)
it means you are waiting for the result and the result will be returned to you, errors must be handled client side ex button to retry

- Asynchronus invocation: S3, SNS, CloudWatch events ecc..
The events are placed in an EventQueue for the lambda to consume, if the vent fails it retries it 3 times, after 1 min and after 2 min
which means your code needs to be Idempotent so it shouldnt break if it retries. We can define a DLQ dead letter queue with SNS or SQS for failed processes

- Event Source Mapping (important?): Kinesis, SQS, DynamoDB, Lambda neds to poll fron the source, it doesnt get invoked. Process streams or Queues (SQS)

Event: Json contains data the function is going to process 
Context: methods and propreties that provide information about the invocation, function, and environment  
<img width="200" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/6344ebcd-3561-4f2a-aae4-1423c294bdb7">

Destinations: Send the results of successful or fails to a destination (Lambda, SNS, SQS, EventBridge)

Lambda Execution Role (IAM Role) Policy to AWS services to ex. event source mapping, permission to Lambda to read from services / write to resources
Lambda Resource base policies: To give other accounts and AWS services to use and invoke Lambda resources

EnvironmentVariables: Key/value pairs in String form, adjust the function behaviour without updating code. Like a config file.

Loggin and monitoring:
CloudWatch logs: lambda logs are stored there if function has the policy
CloudWatch metrics: lambda metrics are displayed there, error counts, invocations, success rates...
X-Ray: Can use in lambda if correct IAM role and enable "Active Tracing", it will run the xray deamon for you

Edge Functions: code attached to CloudFront execution, logic at the edge locations, runs close to users to minimize latency. Deployed globally, serverless
used for: website security, SEO, intelligent Route, real time image transfromation, user authentication, user prioritizing, user tracking and anlytics...
- Cloudfront Functions: JS functions for high scale, latency-sensitive CDN costumization. used to customize the viewer response / requests (before cloudFront
  forwards the request to the origin / after it got the response and returns it to the client). Ex for header manipulation, URL redirects...
- Lambda@Edge: Used to change CloudFront request / responses to both Origin and Viewer requests. higher package dimensions but slower. do what CloudFront functions
  do but with file system access, handles heavier data so it can use the body of the HTTP request, longer execution, both for viewer and origin.

Lambda are launched in a VPC outside of yours, in a VPC managed by AWS so it can't access your VPC resources.
Byt you can deploy it in your own VPC using AWSLambdaVPCAccessExecutionRole
Lambda in a public subnet doesnt have access to internet or public IP, if you want you need to:
By default it doesnt have access to the internet in your private VPC, so you need to give it access with NAT Gateway / instance.
Or you can use a VPC endpoint to privately access AWS services without NAT.  
<img width="300" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/489bec1f-22e1-4e14-9e26-058ed41dc987">

Lambda function configuration:
Ram from 128MB to 10GB, if you need to increase vCPU you NEED to increase RAM, so if your app is CPU-bound, increase RAM!
Tiemout: default 3s, max 15min, if you need more use Fargate, EC2, ECS
Execution Context: temporary runtime that initializer external dependecies / services connection ecc.. is maintained for a short period expecting a new invokation ex:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/b8e7144f-9bdb-4baa-bd52-29d693aa81f5">

/tmp folder if your functiond needs to download a big file, max 10gb of disk space.
if you need persistance space, use S3

Lambda Layers:
custom runtimes for not supported languages, or create a layer to externalize dependencie to reuse them without having to repackage them to deploy, because the lambda can
reference these layers.

File System Mounting:
Can access EFS if running in VPC, its a config to mount EFS file system to local directory at initialization, it usese EFS access points.

Concurrency: the more we invoke the more executions up to 1000, we can set a "reserved concurrency" to limit the max amount per each function.
if we dont set limit, we can have 1 function taking up all 1000 executions, and have the other functions not being able to start.

Cold start and provisione concurrency: new instance so all initializations have to start. The first request has a higher latency, 
you can use a "provision concurrency" to allocate before the function is invoked in advance so cold start never happens.

Dependencies: to include the, you need to zip dependencies and code all together, if > 50 MB upload in S3 and reference it.

Lambda and CloudFormation: 
inline directly in the CF code for very simple functions, we cant include external dependencies.
Through s3, store the Zip in S3 and refer it in CloudFormation code

Lambda Container Images:
Deploy Lambda functions as Container images up to 10BG, pack complex and large dependecies in a container, instead of Zip and upload.

Lambda Versions:
When you work on a function we work on $LATESTS, we we publish we create a version which is immutable and cant be modified (v1,v2 ec...) = code + config.
Aliases used to give the user a stable endpoint, "dev" "prod" ecc... point to the right lambda version. We can also use them for Canary deployment

Lambda and CodeDeploy:
help automate the traffic shift for lamba aliases.
Linear: glow traffic by x percent every n minutes
Canary: x percent then 100%
AllAtOnce: immediate

Lambda function URL:
to expose the function as HTTP endpoint without having to use API gateway ecc... can create a unique URL endpoint, access only with public internet
can have Resource-based policy, CORS security like S3. we can set AuthTypeNone or AuthTypeAWS_IAM for access.

Lambda and CodeGuru: can be acctivate from console to get runtime performance insights

Lamba Limits per region:
128mb-10gb ram memory allocation
max execution 15min
env var. max 4kb
disk capacity /tmp 512MB to 10GB
concurrency ececutions: 1000 (can be increase by support)
compresseed zip deploy: 50MB
size of uncompressed: 250MB

best practices:
never have recursive code.
Heavy duty work put it outside the function handler:
connection to databasees
initialization of AWS SDK
pull in dependencies
Use env. variables for stuff that changes frequently:
DB connection strings
S3 bucket ecc....
Passwords, encrypted with KMS

# DynamoDB
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/60590f72-b4e5-4b77-b546-237a679894c6">

NoSQL serverlesss DB
They can scale horizontally for more Read/Write capacity, do not support joins or operations like "SUM", "AVG"
it a fully managed highly avalible with replica across multiple AZ. 
Scales massive workloads
Fast and consistant performance
Low cost and Auto-Scaling
Standart and IA calsses.

Each table has a Primary Key, infinite rows, each one has attributes can be null or added over time

options to choose the primary key
- Hash: unique for each one like "user_id"
- Partition Key + sort key: "user_id" + "game_id" unique for each item. For example each user can attend multiple games
  so the same partition key is equal in 2 rows but different game_id keys, the combination is unique

Control the R/W capacity modes:
- Provisioned mode (dfault): specify the r/w per second you need to plan your capacity before hand, you pay for what is going to be provisioned
  - RCU read capacity units: rapresent one strongly consistant read per second or 2 eventually consistant read per second, for an item up to 4KB
    - read eventually consistant: if we read right after a write data could be old cuz it hasnt replicated you from the copy we are reading from
    - read strongly consistant: read the data correct for sure, uses TWICE the RCUs
    ex: 10 strongly consistant reads per seond with item 4KB = 10RCUs
    ex: 16 eventually consistant read per second with item size 12KB = (16/2) * (12/4) = 24RCUs
    ex: 10 strongy consistant reads per second with item size 6 KB = 10 * (8/4) because 6 get rounded to 8KB
  - WCU write capacity units: are the throughput, rapresent one write per second for an item up to 1KB, if larger than 1KB, more WCU
    ex: we write 10 items per second with item size 2KB = 20WCUs
    ex: we write 6 items per second, with item size 4.5KB = 30WCUs (the 4.5 gets rounded up to 5)
    ex: 120 items per MINUTE with item size 2KB = 120/60 * 2KB = 4WCUs
  - option to enable auto-scaling to meet demand
  -  throughput can be exceeted temporarely by using "burst capacity"
    
- On-Demand, no provision pay for what you use but more expensive

Data is stored in partitions: primary keys go through hash to know which partition they go to, WCU and RCU are going to be evenly spread across partitions

Throttling: reasons: Hot keys (one partition read too many times ex populat items), Hot partitions, very large items, the solution is Exponential backoff, 
distribute partition keys, if RCU issue use DynamoDB Accelerator DAX

Basic Operations:
- Writing Data:
  - PutItem: create a new item or fully replaces an old one (same primary key)
  - UpdateItem: used to edit a few attributes, but it can also edit all of them, it inserts a new item if not exist
  - Concurrent Writes: the second one overwrite the first one
  - Conditional Writes: accept a Write / update /delete only if conditions are met
    attribute_exist / attribute_not_exist / attribute_type / contains / begins_with / IN and between / size (2 concurrent write could fail because 1 updates and the other
    finds the new data and doesnt match so the second one fails)
  - atomic writes: one says increase by 1, the other increase by 2, at the end the result is 3, both writes succeed
  - batch writes: Write / update many items at a time  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/d5a35a98-dbf1-4cac-a847-e70c3bfcb9bb">  
 
- Reading Data: 
  - GetItem: read based on Primary Key (hash / hash + range), evenetually consistant (default) / strongly, ProjectionExpression to read only certain attributes
  - Query: based on Partition key value (required) , and optional sort key values (=,<,> ec...), FilterExpression additional filtering after query for non-key attributes
  - Scan: get the entire table and filter out on your application (inefficient). supports parallel scan for faster result.
 
- Deleteing Data:
  - Delete an individual item
  - conditional delete
  - DeleteTable
 
- Table Cleanup:
  - scan + deleteItem very slow and expensive
  - drop table + recreate table fast and cheap

- Copy DynamoDbTable
  - use AWS data Pipeline
  - Backup and restore into a new table
  - Scan + PutItem 

 You can batch operations to reduce API calls, it will be done in prallel for efficiency
 - BatchWriteItem: up to 25 PutItem / DeleteItem
 - BatchGetItem: up to 100 items.

PartiQL: do SQL-compatible query (no joins) if all you know is SQL. You can select, CRUD, and supports batch operations.


Local Secondary Index LSI
Alternative sort key for table (same as partition key) based on sort key = scalar attribute (String, Number, Binary) + up to 5 local secondary intexes per table
must be defined at table creation time.
ex: give me all the games that have been played by this user between 2021 and 2023. We have a different sort key thanks to LSI  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4cb1c79c-c587-4ab8-b635-f573ac1fd9c7">  

Global secondary index (GSI)
Alternative Primary Key (hash / hash+range)  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/01541488-4980-4905-8c41-e73febf490a8">  

Optimistic Locking:
"conditional writes" ensure the item hasnt changed before you update/delete it, each item has an atrribute that acts as a version number. 

DynamoDB DAX: Fully managed highly aval. seamless i-memory cache for DynamoDB, fully secure, microseconds cached reads and queries. Doesnt require 
application logic modification because compatible with existing DynamoDB APIs. Solves the "hot key" problem so too many reads. 

DynamoDB Streams: ordered stream of item level modifications (CRUD), streams can be sent to kinesis, Lamba. 
use cases: react to changes in real time ex: send welcome email to users, analytics, insert in derivate tables  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/1f266a95-fd6f-474a-ab9a-2ca2036e5ea4">  
With lambda for table -> DynamoDB Stream -> AWS Lambda event mapping source polls form it

DynamoDB TTL: time to live delete item after an expiry timestamp. Ex to delete session data. It takes up to 48hrs after the TTL for it to get deleted.

DynamoDB Transactions: all or nothing operations. 
In read mode it can be consistance/strong consist./"transactional"
in write mode it can be standart/"transactional"
consumes 2x WCUS and RCU
TransactGetItem one or more GetItem operations
TranctWriteItem ore or more PutItem, UpdateItem, DeleteItem
ex: multiplayer games, financial
ex: 3 transacitonal writes per second with 5KB = 3 * 5 * 2 = 30WCU
ex: 5 transaction reads per second with 5KB = 5 * (8/4)(5 got rounded to 8) * 2 = 20RCU 

DynamoDB Session State cache:
DynamoDB can store sessions is serverless, also ElastiCache can but its in memory. 
At exam if "Session state STORE in meory" = elastiCache, it it talks about automatic scaling = DynamoDB. Also EFS can do the exact same thing but its a file system not DB,
EBS and Instance Store cant be used for shared caching, only local. S3 could be but it has high latency not ment for that small objects

Write sharding:  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/ba8c96fd-4345-459a-92a9-29b125f3344f">  

DynamoDB large objects
max allowed in Dynamo 400kb, we use an s3 bucket to contain a large object  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/4e0d48d0-07df-4c6f-9a3b-949edae5cbd5">  ù
OR  
<img width="400" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/5e20f558-9a14-4fb9-9972-01e02c68b8ff">

Security:
VPC Endpoints to access without internet
Access with IAM
encyption KMS, in trasnit SSL/TLS
backup and restore features
Global tables, fully replicated multi region
DynamoDB Local dev and test apps locally without the offical server
DMS
Can use identity providers / Cognito to exchange credentials for temporary credentiaslw with roles, and can do operations only on data they own

# API Gateway
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/e272307f-1bfe-4156-8f98-dbba2b4c4cd8">

Serverless Expose functions to the world / Access to the app with REST APIs, even with Authentication
It integrates with / can call: 
- Lambda function: invoke / expose the REST API
- HTTP: HTTP endpoints even on premise / ABL (HTTP API)
- AWS Services: Any AWS service API with gateway (HTTP API)
- Websockets: two-way interactive commun. between browser and server, enable stateful apps. server can push info to client
  used in real-time apps like chat, collab platforms, multiplayer games.
  Client connects to API Gateway, it gives a Conncetion ID, lambda uses logic and stores in DB the connection ID,
  it usese a Connection URL Callback to send data to client. You can use Routing to reoute to a specific backend when
  the connection is open (for example to do CRUD different backends) 

3 ways to deploy (called Endpoint Types)
- Edge-Optimized (default): for global clients,  Requests are routed through CloudFront Edge locations (improve latency)
  the API Gateway still lives in only one region
- Regional: for clients in the same regione, could manually combine with CloudFront
- Private: Can only be access from your VPC using an interface VPC endpoint (ENI)

Security: 
Can Authenticate users with:
- IAM roles (for internal apps) create and IAM policy auth and attach to User/Role, IAM credentials are signed
  in the headers.
- Resource policies: Allow for cross account access combined with IAM security, or for specific IP Addresses, or VPC
  endpoint.
- Cognito user pools(for external users) users sign in with congnito first which provides token
- Custom Authorizer (Lambda Authorizer) Token-Bsesd auth. the lambda if auth is correct returns an IAM policy that will
  be cached

- Custom Domain Name HTTPS: security through integration with AWS Certificate Manager (ACM)
  
Deployment Stages:
After Making a change must be deployed it or else changes wont apply, its done through "stages"
each stage has its own config. parameters. Stages are kept and can be rolled back if needed  
<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/8baded8a-55ea-461e-8980-df53eb2cebc8">

Stage variables: like Env. variables (config) but for API Gateway, use them for often changing values
so it prevents redeployment every time. A common use defining to which Lambda function it points
and when you deploy V2 change a % of traffic to the new one changing the Stage Var. (Canary Deployment)

Integration types:
- Mock: API Gateway return a response without sendig the request to the backend
- HTTP / AWS (lambda or AWS services): You must config the integration reqeuest and response, setup data mapping
  with mapping templates for request and response which means we can change the data before it reaches client or backend.
  Mapping template: modify parameters and strings, filter output results, use VTL scripting language for loops,
  if ecc.. We can use it to convert from JSON (REST APIs) to XML (SOAP APIs) 
- AWS_PROXY (lambda proxy) incoming requesta from the client is input data to lambda without changing the data,
  the function handles all logic

It has integration with Open API spec. (its a common way to define REST APIs, using API defined as code.

Can config API Gateway to perform validation so if the data structure doesnt correspond to a scherma returns 400-error
(reduces nr of calls made)

Caching API responses: reduces the nr of calls made, default TTL is 300s, min 0 max 1hr
Caches are different for every stage. We can invalidate the Cache immediatly and flush it, or the user can do that
with max-age=0 if it has the IAM Auth. 

You can make your API aval. for money to customers. We can config how much and fast they can access them, how many calls
possible, use API Keys to identify them, callsers pass the key in x-api-key header. 
Throttling default 10000 rps across all API, in case it happens Error 429 too many req.
Can set Stage limit and Method limit ti increase performance

CloudWatch Logs: Log contains info about request/response body

X-Ray: enable tracing to get extra info

CloudWatch Metrics: Metrics based on stages, CacheHitCount & CacheMissCount, Count (nr of req), IntegrationLatency
(time between request and response from the backend), Latency (time between request and send response to user), 
4XXError (client side) 5XXError (server side)

CORS: must be enable if u want to recieve calls from another domain

# CICD 

Automate and Push code without doing anything else
we want to push code in a repo and deploy it to AWS.

CI: Continious integration, devs will push code to repo, a testing / build checks code, the dev gets feedback and fix bugs
we dont need to test the code its all automatic
CD: Continous delivery: Ensures that the sw can be relased reliably after the test and build. We ensure deploy
happens often and quck, shift away from "one release every 3 months" to "5 releases a day" 

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/7e9b3b94-9646-401f-a808-3bac71924293">

- CodePipeline: automate pipeline. Visual workflow tool to orchestate CICD. We can control the Source / build / test /
  deplot / invoke stages. If a stage fails pipeline stops. You can create events for failed pipelines / events with
  CloudWatch events.
  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/bc65a9a7-7ddf-4d4c-86a5-fe9e9a28747e">  

- CodeCommit: store the code, Versioning control using a version control system. Its fully managed, SSH/HTTPS auth.
  IAM Plicies, encryption at rest with AWS KMS, in transit, 
  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/cc98d861-4d4f-4ab7-90fd-260eda8be962">  

- CodeBuild: Build and test code. The build instructions are in buildspec.yml (must be at root of code like .git).
  Logs can be used in S3 / cloudwatch. we can use CloudWatch Metrics to monitor build statistics.
  Detect faild build and triggers / alarms. 

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f0d2d3dc-08c8-4b10-a775-e59e55911a99">  

- CodeDeploy: automate deploy code to EC2 / lambda (help automate with traffic shift linear / canary) / ECS (only Blue-
  green). Rollback / deploy capability. Gradual deploy control (AllAtOnce, HalfAtATime,OneAtTime,Custom, Blue-green).
  AppSpe.yml file says how deploys shoud happen.  Must run a CodeDeploy Afent on the target instance. 

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/99ca4948-7aae-4adb-9221-4d657205ab0f">  

- CodeStart / CodeCatalyst: Manage sw dev activities.

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/079e00e5-8737-4a92-b317-8b972e9473f7">  

- CodeArtifact: share sw packages, sw packages depend on each other to be built (code dependecncies), storing and
  getting these dependencies is called artifact mangment. We can use Resource policy to auth different packages.

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/8c76f5b0-bfe0-4f1b-8b03-3c983ecfdbe8">  

- CodeGuru: Automated code reivews (reviewer) and app performance (codeGuru profiler) with Machine Learning.

- Cloud9: IDE in clud. can work anywhere in the world if u have internet. 

# SAM
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/f2e46532-12c0-4f01-b0c9-b549809d651e">

Serverless application model basically a shortcut to cloud formation, says how the app should be deployed and should behave
framework for dev and depl. serverless applications.
All config is in YAMAL code, generate complex CloudFormation from simple SAM YAML file
Only 2 commnads to deploy to AWS (sam package and sam deploy)
can use CodeDeploy to deploy lambda functions
Can help you run Lambdda, API Gateway, DynamoDB
AWS toolkits: ide plugin allow you to run build test Lambda functions with AWS SAM.

<img width="500" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/bde587c1-1ff8-47b3-8b14-3d5df83b8d8f">

# CDK
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/181c3088-4372-4996-9a5f-fdc502dfa23c">

Cloud development Kit
define cloud infrastructure using familiar language, code gets compiled into CloudFormation model (Json/Yaml)
You can deploy infrastructure and application code togethere, if it doesnt compile you dont get either of them.
CDK Construct: library collection of constructs for every aws resource.
Construct hub: 3rd party and opne source from community CDKs.

# Cognito
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/17343071-a6ff-4e38-a0f3-a8a9a40080ae">

give user an identity to interact with web or mobile app (outside of AWS) so in exam if they say
"hundreds of users", "mobile users", "authenicate with SAML"

Cognito user pool (CUP): create a serverless DB of your users. sign in function for app users. 
Username / email / password / MFA / phone verif. / login with google (federate identity). block users compromised. 
when they login they get back a JSON web toke JST. 
CUP can invoke Lambda on triggers. 
It offers a Hosted autentication UI that you can use in your apps.
Offers Adaptive Authentication: block sign-ins or require MFA if the login is suspicious.
We can integrate it with API Gateway and ALB (offload the auth. to the load balancer) we can use 2 ways to auth:
With Cognito user pools, or OIDC auth.

Cognito Identity pools (federated identitites): AWS credentials to users to access AWS resources directly, temporary
credentials by logging in with public providers (amazon, google, apple, facebook), or users in an Amazon Cognito pool.
we can also allow for unauthenticaded gues acccess.  
<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/9af0fe7d-94c5-4a2b-a5fe-c132c533bf99">

# Step functions  
<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a419bce9-9dd6-4b93-92fd-59d51c9d550a">

perofrm many simple tasks.
Model your workflow as state machines (one per workflow) for order fulfillment, data processing, web app... any workflow
Writte in JSON, visualized. In task states you can invoke AWS services, like lambda, do dynamoDB stuff ecc...
States can be: Choice (test condition) / fail or succeed / Pass (pass data with no work) / Wait / Map state /
PARALLEL STEP - begin parallel brancesh of execution.

<img width="300" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a63f3b08-3aa7-44be-b5b7-92d1dba4535b">

Error handling: we can have retry or catch (transit to failure path)

Wait for task token: allows you to pause step function until a task token is returned. by appending .waitForTaskToken

Activity task: allows you to have the task work performed by an Activity worker (Apps on EC2 / Lambda / mobile device...)
after it send a SendTaskSuccess or SendTaskFailure. To set how long it should run TiemoutSettings, periodic send beat
with SendTaskHeartBeat 

2 types of workflows:
- Standard (default):
max duration 1 year
price by nr of states
used for non-idempotent actions (same request leads to the same system state, and no action is unintentionally 
executed more than once) like payment process.

- Express:
  up to 5 min, higher capacity, billed by nr of executions, duartion and memory consumption. used for IoT data ingestion,
  streaming Data, mobile app backends.
  - Async (At least once execution guarantee) dont wait for result, ex message services you dont wait for response.
    must manage indepontance to manage that if it runs twice you dont have twice the effects.
  - Sync (At most once execution guarantee) Wait for workflow to complete. When you need an immediate response. At-most
    because if there is a failure it doesnt automatically restart, its up to your logic.

AWS App sync:  

<img width="50" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/c86141c0-adb0-4b08-8784-c5ced50208ef">  

managed service that useses GraphQL: makes it easy for app to get exactly the data they need. You ask for a field
and that is what gets returned. You can combine data from multiple data sources types, noSQL, SQL, HTTP, APIs...
You can also use it to get real-time data with WebSocker or MQTT on WebSocket, so if u need a field real time
For mobile apps: can be used for local data access and data synchronization.
To start upload a GraphQL query
Security: API_KEY, AWS_IAM, OPENID_CONNECT, AWS_COGNITO_POOLS, Https with CF in front.

AWS Amplify:
EB for mobile and web apps. Gives us Data Storage, auth, ml, fontend libraries.
gives you Authentication out of the box with cognito.
gives you Data Store out of the box with AppSync and DynamoDB

<img width="450" alt="image" src="https://github.com/99ionut/AWS-Certified-Developer-Associate-DVA-C02-Study-Notes/assets/73752549/a3f6a3a4-f326-4f00-a43c-4a1af6226434">























  



















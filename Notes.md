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
- SDK (access keys)
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
 
# EC2 STORAGE
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

# Amazon EFS
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
<img width="360" alt="image" src="https://github.com/ionutsuciu1999/AWSnote/assets/73752549/d48cb9cf-8a74-4eb9-a5ba-182dd28d3a23">

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
Used for everything, is the backbone for a lot of internet stuff
backup and storage, disaster recovery, archive, hybrid cloud storage, app hosting, media hosting, data lakes and big data, sw delivery, static website

store objects (files) in "buckets"
must have unique name
defined at the region level ( not global level)

contains:
- each object has a key: the full path of the file, prefix + object name
- max size is 5TB, files > 5gb are multi part upload
- metadata list of key-val per object
- tags key-val for security / lifcycle
- version ID (if versioning enabled) 

security:
- IAM policy for users
- Bucket policy; bucket wide rules at the resource level
- object ACL
- bucket ACS
- encrypt the object with encryption keys

S3 can host static websites and have the accessible public with a policy

you can version your files, its enabled at the bucket level, instead of overwiting it creates a new version

replication:
enable versioning 
- CRR cross region replication (compliance or low latency)
- SRR same region replication (live replication, log aggregation)
- buckets can be in diff AWS accounts
- copy is async


  




















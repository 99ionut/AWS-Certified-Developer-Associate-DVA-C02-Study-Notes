# IAM
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
  - "Contition": when its applied (optional)

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

Amazon EFS
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

#ELB
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
- scaling capability / Auto scaling
- storage backed by EBS























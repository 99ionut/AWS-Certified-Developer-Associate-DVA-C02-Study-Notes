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
- EBS (elastic block store) network drive you can attach to your drive while they run
  - persistant data, only mounted to 1 ec2 at the time, in only 1 AZ at the time
  - network drive so a bit latency to communicate (not phisycal drive)
  - because its network it can be detached and reattached somewhere else
  - can have a delete on termination attribute




















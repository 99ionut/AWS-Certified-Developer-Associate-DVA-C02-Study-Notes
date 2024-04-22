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
-"version": the policy version
-"Id": id for the policy (optional)
-"Statement": required
  -"Sid": identifier (optional)
  -"Effect": allow/deny access to certain APIs
  -"Principal": account/user/role its applied to
  -"Action": API calls that will be allowed / denied based on the effect
  -"Resource": list of res. to which the action is applied
  -"Contition": when its applied (optional)

# EC2

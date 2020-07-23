# Ultimate AWS Certified Cloud Practitioner - 2020 (notes)
- [Ultimate AWS Certified Cloud Practitioner - 2020 (notes)](#ultimate-aws-certified-cloud-practitioner---2020-notes)
  - [General notes](#general-notes)
  - [Section 3](#section-3)
  - [Section 4](#section-4)
  - [Section 5](#section-5)
  - [Section 6](#section-6)
  - [Section 7](#section-7)
  - [Section 8: S3](#section-8-s3)
  - [Section 9: Databases & Analytics](#section-9-databases--analytics)
  - [Section 10: Other Compute Services: ECS, Lambda, Batch, Lightsail](#section-10-other-compute-services-ecs-lambda-batch-lightsail)
  - [Section 11: Deployments & Managing Infrastructure at Scale](#section-11-deployments--managing-infrastructure-at-scale)
  - [Section 17: Machine learning](#section-17-machine-learning)
  - [Section 19: Advanced identity](#section-19-advanced-identity)

## General notes

- See `REVIEW` for points to review

## Section 3

AWS
- region -> AZ (one or more data centers)
- edge locations: are not data center, just content delivery
- IAM and some others are global
- pin services to console bar
- region table

## Section 4

- groups can't contain groups
- a user belongs to 0 or more groups ; best practice at least one group
- a user has policies (json doc)
- example policy JSON:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "iam:SimulateCustomPolicy",
        "iam:Get*"
      ],
      "Resource": "*"
      }
    ]
}
```
- password policies:
  - minimal length/char types/etc.
  - allow the user to change the pwd
  - require to change after some time
  - prevent pwd reuse
- MFA
  - virtual device: google authenticator, authy
  - U2F key (eg. YubiKey)
  - Other HW device
- AWS access options
  - console
  - CLI
  - SDK
- IAM roles: assign permissions to AWS services
- IAM security tools
  - IAM Credentials report (account-level): very detailed list of users
  - IAM Access Advisor (user-level): show service permissions granted to user and last usage
    - activity generally appears after 4 hours

## Section 5

- billing permissions are granted through a checkbox in the root account settings
- EC2: security groups=firewall, startup scripts=user data
  - SGs have only "allow" rules
  - SG rules editing has a convenient `My IP` source
- EC2 Instance Connect: creates the `ec2-user` user; may work only on "Amazon Linux 2" AMI
- Instance purchase options:
  - On-demand
    - payment: Linux=per second, after first minute; Other=per hour
  - Reserved (1 year min)
    - Convertible: have flexible attributes (e.g. instance type) that can be exchanged
    - Scheduled: time-scheduled
  - Spot: cheaper, but can lose instances, if the max price set is less than the current spot sprice (which fluctuates)
    - best for workloads resilient to failure: batch/processing jobs, distributed workfloads, etc.
  - Dedicated host (3 years)
    - can be used for compliance requirements/licensing simplifications
  - Dedicated instance: similar to D.H., but there's no ownership/visibility of the HW (eg. can change after stop/start)

## Section 6

- An EBS volume is bound to an AZ
- EBS volumes are networked, so there is (minor) latency
- It's recommended to take snapshots on detached volumes
- Snapshots (actions):
  - can be copied to any region
  - can be converted to volumes, in any AZ of the same region
  - can be converted to images
- AMIs are per-region (but can be copied across regions)
  - "Public AMIs" are provided by AWS; "Marketplace" is for 3rd party
- EC2 instance store: EC2 instances with locally attached drivers
  - WATCH OUT! Lose their storage when stopped
- EFS works across AZs
  - more expensive than gp2; pay per use; no space limits

## Section 7

- Scalability: (conventional)
- Elasticity: AWS implementation of horizontal scalability
- Agility: Distractor: simple UX

- LBs
  - Application: HTTP(S), Layer 7
  - Network: TCP/UDP/TLS Performance, Layer 4
  - Classic: Obsolete, mix of the two

- LBs can handle: instances, IPs, and lambdas

- An LB requires two subnets

## Section 8: S3

- Bucket names must be *globally* unique, but they're defined at region level
- Naming: no uppercase, no underscore, 3-63 chars long, no ip, must start with letter/number
- Prefix: essentially, the path; key: essentially, the base filename

- Objects have:
  - max size: 5 TB; files larger than 5 GB must use "multi-part" upload
  - metadata
  - (max 10) tags
  - version id

- Security
  - User based (IAM policies)
  - Resource based
    - Bucket policies
      - Bucket setting (includes encryption)
      - JSON
    - Object ACLs
    - Bucket ACLs (less common)
  - Encryption

- A principal can access an object if it's explicitly granted and it's not denied

- Cross-account access: use bucket policy

- Example policy: (WRITEME)

```json
{
  "Id": "Policy1595497452733",
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1595497451256",
      "Action": [
        "s3:GetObject"
      ],
      "Effect": "Allow",
      "Resource": "arn:aws:s3:::saverio-test/*",
      "Principal": "*"
    }
  ]
}
```

- the ARN can be easily found in (bucket) -> `Permissions` -> `Bucket Policy`
- don'e forget to suffix `/*` to apply policy to the objects!

- Website (property) link:
  - `$bucket_name.s3-website-$aws_region.amazonaws.com`
  - `$bucket_name.s3-website.$aws_region.amazonaws.com`

- Versioning
  - Is at bucket level
  - It's best practice to activate it
  - Objects stored before enabling have `null` version
  - When an object is deleted, a pseudo-object "Deletion marker" is placed; delete it in order to restore the file

- Logging
  - Requires versioning on both buckets
  - WATCH OUT! Don't select the same bucket as logging; it will create an infinite cycle and cost a lot of $$$

- Storage classes
  - Standard (99.99% = 53'); stored on 3 AZs
  - Infrequent access (IA) (99.9%); still immediate to access; stored on 3 AZs
  - One-zone IA (99.5%); cheaper and less available version
  - Intelligent tiering: automatic between Standard and IA, based on rules
  - Glacier; minutes to hours retrieval
  - Glacier deep; hours retrieval
  - Deprecated: Reduced Redundancy Storage

- S3 id not 100% durable!!!!!!! The is a chance to lose an object, although it's very small
- Availability expected: 99.99% = 53 minutes downtime per year

- Snowball (deprecated), Snowball Edge (with compute), Snowmobile (MWAHAHA)

- Storage gateway: very vague; intended to support hybrid cloud

## Section 9: Databases & Analytics

**watch out**: questions don't use rigorous a "database" definition.

- RDS is backed by EBS (`gp2`/`io1`)
- Aurora supports both MySQL and PGSQL
  - Storage grows automatically
  - More expensive than RDS
- DynamoDB: 3 AZs
  - On-disk
  - Very low latency and huge scaling
  - Autoscales
  - Has a PK (one or multiple keys), and multiple attributes
- Redshift: PGSQL for OLAP
  - Load data once every hour, not second (!?)
  - Columnar
  - Highly parallelized and scalable
- EMR: Elastics Map Reduce (Hadoop)
  - Handles clusters of hundreds of EC2 instances
  - Autoscales; integrates with Spot instances (!)
- Athena (serverless): queries S3 and stores results in it (!)
  - supports SQL
  - per-query payment

- Data Migration Service (DMS)
  - runs on an EC2 instance
  - supports both homogeneous/heterogeneous migrations (same/different db product)

- Glue (serverless)
  - ETL service: prepare and transform data, eg. S3+RDS -> Redshift
  - Has a "Glue Data Catalog" that can be used by other services to know the data store schemas

## Section 10: Other Compute Services: ECS, Lambda, Batch, Lightsail

- ECR: Elastic Container Registry: private Docker repository (to be used with ECS/Fargate)
- ECS: Elastic Container Service: manages running of containers on top of EC2 instances (but the instances need to be managed)
- Fargate: Serverless ECS
- Lambda is event-driven, paid per RAM and time
  - Lambdas require a role
- Batch: like Lambda, but one provides the containers; doesn't have all the Lambda limits
- Lightsail: Simplified version of AWS

## Section 11: Deployments & Managing Infrastructure at Scale

- Cloudformation: tagging for costs (?), timed operations!, diagram
  - Stack Designer
    - has templates (ready, and YAML- uploadable)
- Beanstalk: Platform as a Service; essentially, preconfigures app+infrastructure
  - free, but underlying instances are paid
  - 3 models
    - Single instance: for dev
    - LB + ASG: traditional webapps
    - ASG: workers-style
  - Many plarforms/languages, including custom ones
- CodeDeploy
  - Can upgrade both EC2 instances and on-premises machines (=hybrid)
  - Requires the CD agent
- Systems Manager (SSM), also hybrid: sort of managed Capistrano for systems maintenance
  - Patch, run commands, stores configuration
  - Requires the SSM agent
- OpsWorks: Chef/Puppet

## Section 17: Machine learning

- Rekognition: ML for finding/recognizing objects in images/videos
- Transcribe: Deep learning ("ASR") for speech into text
- Polly: opposite of Transcribe
- Translate
- Lex & Connect: chatbots
  - Lex: backend for call centers etc. (also Alexa); uses ASR+Natural Language Understanding (NLU) to serve the request
  - Connect: frontend for call centers etc: takes calls, define workflows
- Comprehend: NLP service to understand (process) texts
  - For example, use to mass-analyze/categorize articles, requests...
- SageMaker: managed service for building Machine Learning models

## Section 19: Advanced identity

- Cognito: Large-scale authentication (including 3rd party like Google/Facebook), including mobile apps
- Active Directory support: Simple/Managed AD, Trusted (work together), and Connector (proxy)
- SSO: Amazon's login service
  - exam keyword: business applications
  - exam: manage multiple accounts, but not business applications -> Organizations


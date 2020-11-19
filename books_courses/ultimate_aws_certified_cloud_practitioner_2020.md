# Ultimate AWS Certified Cloud Practitioner - 2020

- [Ultimate AWS Certified Cloud Practitioner - 2020](#ultimate-aws-certified-cloud-practitioner---2020)
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
  - [Section 12: Leveraging the AWS Global Infrastructure](#section-12-leveraging-the-aws-global-infrastructure)
  - [Section 13: Cloud integrations](#section-13-cloud-integrations)
  - [Section 14: Cloud monitoring](#section-14-cloud-monitoring)
  - [Section 15: VPC & Networking](#section-15-vpc--networking)
  - [Section 16: Security & Compliance](#section-16-security--compliance)
  - [Section 17: Machine learning](#section-17-machine-learning)
  - [Section 18: Account Management, Billing & Support](#section-18-account-management-billing--support)
  - [Section 19: Advanced identity](#section-19-advanced-identity)
  - [Section 20: Marketese super-trash](#section-20-marketese-super-trash)

## General notes

- See `REVIEW` for points to review
- Scrub `64kramsystems.com` and `saverio`s
- How to create account alias?

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

## Section 12: Leveraging the AWS Global Infrastructure

- Global: for latency

- R53 Routing policies:
  - simple
  - weighted
  - latency
  - failover, with health check on the designated primary

- Cloufront
  - caches (on/from first access), for a TTL (maybe a day); origin: S3 or HTTP
  - enhanced security: via CF Origin Access Identiy (OAI) (added to bucket policy), DDoS protection (AWS Shield), WAF
  - CF can be used as ingress to upload files to S3

- CF <> x-region replication
  - CF has more edges (200+)
  - XRR is updated almost-realtime
  - XRR has lower latency
  - CF is not read-only

- CF creation procedure (1)
  - this time, block the public access on the bucket
  - create distro: web
  - select the bucket
    - even lbs can be chosen
  - restrict bucket
  - OAI: create new identity
  - grant read (update bucket policy)

- CF: the http address can be found in `Domain Name`

- CF creation procedure (2) -> regional, for faster propagation
  - in the origin, add the region
    - e.g. `sav986-private.s3.amazonaws.com` -> `sav986-private.s3-eu-central-1.amazonaws.com`
    - otherwise, access will get 307, during propagation
      - see https://aws.amazon.com/premiumsupport/knowledge-center/s3-http-307-response
    - also restrict access, use existing identity, update bucket policy
  - access is now available (check the address)
  - one can see the new policy in the bucket

- S3 Transfer Acceleration
  - Uses edge locations to upload files to S3, in order to speed up the upload

- Global Accelerator
  - Same as S3 TA, except for applications
  - Connection through "Anycast IP"s

## Section 13: Cloud integrations

- Simple Queue System (SQS)
  - Pull system: messages are put in a queue, and the consumers pull them (1 consumer per message)
- Simple Notificaation System (SNS)
  - Push: messages are sent, with a topic assigned, and subscribers to the topic are sent the message
  - Subscribers can be many services, including Lambda, email etc.

## Section 14: Cloud monitoring

- CW Metrics
  - Reported every 5 minutes (!!); extra payments for reporting every minute
  - Billing estimated charge, only in `us-east-1`
  - EC2 RAM not available!
  - API metrics available
  - Custom metric available

- CW Actions: auto scaling, EC2 actions, SNS notifications...

- CW Logs: captures logs from many sources
  - including: CloudTrail, Elastic Beanstalk, Route53, etc.
  - Agent needs to be installed in order to push the log files
    - Agent can also be installed on on-premises host
  - Groups: are automatically created for some services, e.g. Lambda

- EventBridge (formerly CW Events): inputs from many sources
  - AWS services (scheduled, resource events...)
  - Partner services: Zendesk, etc.
  - Custom Event buses
  - Schema Registry

- CloudTrail: logs all the events
  - enabled by default
  - sources: API, SDK, CLI, Services
  - output to S3/CW logs

- X-Ray: Visual analysis of application performance and (some) errors

- Health dashboards:
  - `Service`: Global status of services
  - `Personal`: Related to services that may have an impact on one's infrastructure

## Section 15: VPC & Networking

- Subnets are inside AZ; can be private or public (with an IGW)
  - private can access internet via NAT GW)

- NACLs: operate at subnet (IPs) level; have allow/deny; there is a default one; stateless
- Security groups: operate at EC2 instance level; have only allow; stateful (outgoing traffic is allowed if originates from incoming)

- VPC peering: allow VPCs to communicate; CIDRs must not overlap; not transitive!
- Transit gateway: peering on large scale, hub-and-spoke architecture

- Flow logs: storage on CW or S3

- Hybrid networking
  - Site-to-site VPN: requires customer GW and Virtual Private GW
  - Direct Connect: physical (expensive): fast and private

- VPC endpoints: connect services to private subnet
  - GW: S3/DynamoDB
  - Interface: the rest

## Section 16: Security & Compliance

- Shared responsibility exam keywords: "Path Management", "Configuration Management", "Awareness & Training"

- WAF
  - Layer 7 (Application, highest)
  - Deploy on Application LBs, API Gateway, Cloudfront
  - Web ACL (rules), including packet origin and request rate

- Shield: Layer 3/4; two levels
  - Basic, free, enabled by default
  - Advanced
    - 3k/month/org
    - DDoS and more sophisticated attacks
    - Spike fees absorbed by AWS
    - Special support

- Safe design
  - Route 53 (-> Shield)
    - CF (-> WAF)
      - VPC (-> subnets, NACL, SGs)

- Penetration testing: all mass-operations (eg. scanning, flooding) are prohibited

- KMS encrypted services by default
  - Cloutrail logs
  - Glacier
  - Storage GW

- KMS keys
  - AWS-managed (used by services)
  - Customer-managed
  - HSM: expensive, HW solution, for clients deploying their own keys

- Secrets Manager
  - Uses KMS
  - Mostly meant to be integrated with RDS

- Artifact: Produce documentation useful for assessments/certifications/etc.

- Inspector: Analyzes EC2 instances for vulnerabilities (requires agent)

- Macie: Analyzes S3 data for fiding personal data

- Config: Stores, reviews and notifies about configuration history of AWS resources
  - Configured with rules (e.g. port 22 open)
  - Per-region

- Guardduty (per-region)
  - Analyzes logs: VPC Flow, CloudTrail, DNS
  - Generates CW Events

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

## Section 18: Account Management, Billing & Support

- Organizations
  - Two types: "full" and for consolidated billing only
  - Advantages (full)
    - Consolidated billing
    - Volume savings
    - Pooling of reserved instances
    - API for creating accounts
    - Have additional security layer (Service Control Policies)

- Typical design important: central account for logging

- Organizational Units (OUs)
  - Can be multi-level nested
  - 1 OU per account
  - Inherit SCPs

- Organizations: there is one master account; can be in any OU

- Reservations
  - commitment to payment per hour to instance family
  - independent of AZ, size, OS or tenancy

- Payment model
  - EC2: Linux=per second, Windows=per hour
  - Fargate: per vCPU+memory
  - S3
    - no payment for inbound traffic
    - payment also for: lifecycle transition, acceleration
  - CF: different per region
  - EC2 networking (scale of expense)
    - via public IP
    - across regions
    - across AZs
    - no payment for inbound traffic

- Billing tools
  - `TCO`: Estimate: On-premises vs AWS
  - `Pricing calculator`: Estimate: Cost of infrastructure (insert the components)
  - `Billing (+Free tier) dashbord`: Very high-level overview
  - `Cost and usage reports`: In-depth reports (csv-style)
  - `Cost Explorer`: Review costs (over time)
    - includes `Savings Plan`
    - includes `Forecast`
    - can send reports (at intervals)
  - `CloudWatch`: Alarms-oriented, simple
  - `Budgets`: Alarms-oriented, complex

- Cost allocation tags
  - added by AWS: `aws:`, added by user: `user:`
  - can be added via Tag Editor
  - aggregated into Resource Groups

- Trusted advisor: performs lots of different high-level checks, and gives advice
  - Free tier has limited checks

- Support plans (additions)
  - Basic: 24/7 support (what?), email (support associates), one contact
  - Developer: Email (support engineers)
  - Business: Phone, Full Trusted advisor+API, Production impaired SLA
  - Enterprise: Account manager, Concierge, More production SLA

## Section 19: Advanced identity

- Cognito: Large-scale authentication (including 3rd party like Google/Facebook), including mobile apps
- Active Directory support: Simple/Managed AD, Trusted (work together), and Connector (proxy)
- SSO: Amazon's login service
  - exam keyword: business applications
  - exam: manage multiple accounts, but not business applications -> Organizations

## Section 20: Marketese super-trash

- Operational excellence
  - Principles
    - Infrastructure as code
    - Documentation
    - Small, frequent changes
    - Improve operations
    - Anticipate failures
  - Services: CloudFormation, Config, CloudTrail/CloudWatch, X-Ray, Code*
- Security
- Reliability
  - Principles
    - Recovery
    - Scale horizontally/Don't guess capacity
    - Automate change
  - Services: IAM, VPC, Service Limits, Trusted Advisor, Auto Scaling, CloudTrail/CloudWatch, Config, Backups, CloudFormation, S3, Route 53
- Performance Efficiency
  - Principles
    - Democratize advanced techs
    - Go global in minutes
    - Serverless
    - Experiment more often
    - Mechanical sympathy WTF
  - Selection: a lot
- Cost Optimization

- Note: Downtime can be a disadvantage of vertical scaling

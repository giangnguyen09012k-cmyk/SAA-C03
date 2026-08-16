
# **IAM — Identity & Access Management**

**What it is:** IAM is the global service that controls WHO can do WHAT to WHICH AWS resources. It is free, global (not region-specific), and foundational to every AWS architecture.

## **Core IAM Components**

| Component | What it is | Real-World Analogy | Key Exam Fact |
| ----- | ----- | ----- | ----- |
| User | A permanent identity for a person or application | An employee badge | Has long-term credentials (password \+ access keys) |
| Group | A collection of users sharing the same permissions | A department (HR, Dev, Finance) | Groups cannot be nested; policies attach to groups, not directly to users best practice |
| Role | A temporary identity assumed by services, users, or external accounts | A visitor badge you wear and return | No long-term credentials; uses STS tokens (15 min–36 hrs) |
| Policy | JSON document defining Allow/Deny for actions on resources | A rulebook/terms of service | Explicit Deny always wins over any Allow |
| STS | Security Token Service — issues temporary credentials | A temp-pass vending machine | AssumeRole, AssumeRoleWithWebIdentity, GetFederationToken |

## **IAM Policy Evaluation Logic**

AWS evaluates policies in a strict priority order. Knowing this order is critical for the exam.

| Priority | Evaluation Step | Result |
| ----- | ----- | ----- |
| 1 (highest) | Explicit Deny in ANY policy | DENY — game over, no exceptions |
| 2 | Organizations SCP (Service Control Policy) | If SCP doesn't Allow → DENY |
| 3 | Resource-based policy (e.g. S3 bucket policy) | May grant cross-account access |
| 4 | Identity-based policy (IAM policy on user/role) | Explicit Allow needed |
| 5 | Permission Boundary | Must also Allow what identity policy allows |
| 6 (lowest) | Session Policy (when assuming roles) | Further restricts temporary session |

| ⚠️ EXAM TRAP: No explicit Allow \= implicit DENY. A user with no policies attached cannot do anything at all. This trips up many candidates who assume a blank slate allows read-only access. |
| :---- |

## **IAM Roles — The Most Important IAM Concept for SAA**

Roles are used constantly in SAA exam scenarios. Learn to recognize WHEN to use a role vs other options.

| Scenario | Solution | Why Not Alternatives |
| ----- | ----- | ----- |
| EC2 instance needs to read S3 | Attach IAM Role to EC2 instance profile | Never store access keys on EC2 — they can be stolen from metadata |
| Lambda needs to write to DynamoDB | Execution Role on Lambda function | Same reasoning — roles provide temp creds automatically |
| Account A needs to access Account B's S3 | Role in Account B; Account A users AssumeRole | Cross-account access \= Roles, not copying credentials |
| Mobile app users (millions) need AWS access | Cognito Identity Pool \+ Web Identity Federation | Cannot create IAM user per customer — doesn't scale |
| On-prem employees need AWS access via AD | AWS SSO / IAM Identity Center with AD connector | Federation with SAML 2.0 — no AWS user creation needed |
| Third-party audit needs read-only access | Role with External ID (Confused Deputy protection) | External ID prevents one tenant impersonating another |

## **IAM Policies: Inline vs Managed**

| Type | Description | When to Use | Exam Tip |
| ----- | ----- | ----- | ----- |
| AWS Managed Policy | Pre-built by AWS (e.g. AmazonS3ReadOnlyAccess) | Starting point, common use cases | Cannot modify; may be overly permissive |
| Customer Managed Policy | You create and maintain; reusable across entities | Production environments, least privilege | Version-controlled; best practice for shared policies |
| Inline Policy | Embedded directly in one user/role/group; deleted with entity | Strict 1:1 relationship needed | Hard to audit; generally discouraged |

## **Permission Boundaries**

**What it does:** Sets the MAXIMUM permissions an IAM entity can have. The effective permissions are the intersection of the identity policy and the permission boundary.

**Use case:** You want to allow developers to create IAM roles/users for their services, but ensure they cannot grant more permissions than they themselves have (prevents privilege escalation).

| 💡 TIP: Think of Permission Boundary as a fence. The identity policy says what they CAN do. The boundary says what the fence allows. Only actions inside BOTH the fence AND the policy are permitted. |
| :---- |

## **MFA & Security Best Practices**

| Practice | Details |
| ----- | ----- |
| Root Account | Enable MFA immediately; never use for day-to-day tasks; delete access keys |
| MFA Delete on S3 | Requires MFA to delete S3 objects or change bucket versioning — needs root account to enable |
| Access Key Rotation | Rotate every 90 days; use IAM Access Analyzer to find unused keys |
| Least Privilege | Start with minimal permissions and add as needed; use Access Advisor to see what's used |
| IAM Access Analyzer | Identifies resources shared outside your org/account (unintended public exposure) |

| 🧠 MNEMONIC: GRUELS for IAM entities: Groups, Roles, Users, External IDs, Limits (boundaries), STS. Everything in IAM connects to these six. |
| :---- |

## **IAM Roles Anywhere**

| Feature | Detail |
| ----- | ----- |
| **Purpose** | Securely grants temporary AWS IAM role-based credentials to servers, containers, or applications running outside of AWS (on-premises or in other clouds). |
| **How it Works** | Uses public key infrastructure (PKI) certificates issued by your local Certificate Authority (CA) to authenticate. By establishing a trust relationship between your CA and AWS IAM Roles Anywhere, on-premises clients can call the AWS Security Token Service (STS) to assume an IAM role. |
| **Why Use It** | Eliminates the security risk of storing long-term, hardcoded AWS Access Keys (`AccessKeyId` and `SecretAccessKey`) on physical on-premises servers. |

## **AWS Directory Service & AD Connector**

| Feature | Detail |
| ----- | ----- |
| **AD Connector** | A directory gateway that redirects directory requests to your on-premises Microsoft Active Directory without caching or replicating any data in the cloud. |
| **Use Case** | Best when you want to enable on-premises employees to log into the AWS Management Console and access AWS resources (like Amazon WorkSpaces or FSx for Windows File Server) using their existing Active Directory corporate credentials. |

<br>

# **EC2 — Elastic Compute Cloud**

EC2 is virtual machines in the cloud. The SAA exam tests your ability to choose the RIGHT instance type, purchasing option, and surrounding architecture — not just that EC2 exists.

## **EC2 Instance Families — Know Which to Pick**

| Family | Optimized For | Use Cases | Memory Aid |
| ----- | ----- | ----- | ----- |
| General Purpose (T, M) | Balanced CPU/Memory/Network | Web servers, dev/test, small DBs | T \= Tiny/Burstable; M \= Most balanced |
| Compute Optimized (C) | High CPU | Batch processing, HPC, gaming servers, ML inference | C \= CPU crunching |
| Memory Optimized (R, X, z, u) | High RAM | In-memory DBs (Redis), SAP HANA, big data analytics | R \= RAM; X \= eXtra RAM |
| Storage Optimized (I, D, H) | High I/O throughput | NoSQL DBs, data warehousing, Hadoop, OLAP | I \= IOPS; D \= Dense storage |
| Accelerated (P, G, F, Inf, Trn) | GPU / FPGA / AI chips | ML training, video transcoding, HPC, genomics | P/G \= GPU; Inf/Trn \= AI chips |

## **EC2 Purchasing Options — THE Most Common SAA Decision**

This is tested constantly. Match the scenario to the cheapest/correct purchasing option.

| Option | Discount vs On-Demand | Commitment | Best For | Exam Keyword |
| ----- | ----- | ----- | ----- | ----- |
| On-Demand | 0% (baseline) | None | Short-term, unpredictable workloads | "short-term", "unpredictable", "testing" |
| Reserved (Standard) | Up to 72% | 1 or 3 years | Steady-state predictable workloads | "steady state", "always running", "known load" |
| Reserved (Convertible) | Up to 66% | 1 or 3 years | Same but need to change instance type | "need flexibility", "might change instance" |
| Savings Plans (Compute) | Up to 66% | 1 or 3 years | Flexible: auto applies to EC2, Fargate, Lambda | "most flexible", "Lambda \+ EC2 mix" |
| Spot Instances | Up to 90% | None (can be interrupted) | Fault-tolerant, batch, stateless workloads | "fault tolerant", "batch", "cheapest" |
| Dedicated Hosts | On-Demand price or Reserved | Optional 3yr | Compliance, bring-your-own-license (BYOL) | "BYOL", "compliance", "per-socket licensing" |
| Dedicated Instances | \~10% premium | None | Hardware isolation without managing host | "hardware isolation", "not share with others" |
| Capacity Reservations | On-Demand price | None | Guarantee capacity in specific AZ | "guaranteed capacity", "critical workload" |

| ⚠️ EXAM TRAP: Dedicated Hosts vs Dedicated Instances: Hosts give you control of the physical server (socket/core visibility for BYOL). Dedicated Instances just mean your VMs don't share hardware with other AWS customers — but you don't control the host. |
| :---- |

| 💡 TIP: Spot Instance interruption: AWS gives a 2-minute warning via instance metadata (spot/termination-time) and EventBridge. Design workloads to checkpoint state and handle interruption gracefully. |
| :---- |

## **EC2 Storage Options**

| Storage Type | Type | Persistence | Max IOPS | Use Case | Key Facts |
| ----- | ----- | ----- | ----- | ----- | ----- |
| EBS gp3 | Block | Persistent | 16,000 | General purpose OS volumes | Baseline 3,000 IOPS; independently provision IOPS and throughput |
| EBS io2 Block Express | Block | Persistent | 256,000 | Critical databases, Oracle, SAP | 99.999% durability; Multi-Attach (up to 16 instances) |
| EBS st1 | Block | Persistent | 500 | Sequential large file throughput | Throughput-optimized HDD; NOT bootable |
| EBS sc1 | Block | Persistent | 250 | Cold infrequent data | Cheapest HDD EBS; NOT bootable |
| Instance Store | Block | Ephemeral | Millions | Temp buffer/cache | Lost on stop/terminate; fastest possible I/O |
| EFS | File (NFS) | Persistent | 500K+ | Shared filesystem across instances | Multi-AZ, auto-scales, Linux only |
| FSx for Windows | File (SMB) | Persistent | High | Windows shared folders, Active Directory | Managed Windows File Server; Active Directory integration |
| FSx for Lustre | File (POSIX) | Persistent/Scratch | Millions | HPC, ML, financial modeling | Can link to S3; sub-ms latency |
| FSx for NetApp ONTAP | File (NFS & SMB) | Persistent | High | Legacy migrations needing both NFS and SMB | Simultaneous multi-protocol access; Active-passive clustering across Multi-AZ for HA |
| FSx for OpenZFS | File (NFS) | Persistent350,000+| High-performance |Linux workloads, web serving, dev/testSub-millisecond latency | OpenZFS features (instant snapshots, zstd compression, cloning) |

| 💡 TIP: EBS Multi-Attach (io1/io2 only): Multiple EC2 instances in the SAME AZ can attach the same volume. Used for clustered databases like Oracle RAC. Each instance must manage concurrent writes — not automatic. |
| :---- |

## **EC2 High Availability & Auto Scaling**

For the exam, you'll design HA architectures. Understand how these components interact.

### **Auto Scaling Group (ASG) Policies**

| Policy Type | How it Works | Best For | Exam Scenario |
| ----- | ----- | ----- | ----- |
| Target Tracking | Maintains a metric at target (e.g. CPU \= 50%) | Most workloads; simplest to configure | "keep CPU at 40%" — choose Target Tracking |
| Step Scaling | Adds/removes capacity in steps based on alarm thresholds | Tiered response to demand spikes | "add 2 at 70%, add 4 at 90%" — choose Step |
| Simple Scaling | One action per alarm (waits for cooldown) | Legacy; avoid for new designs | Old style; Step Scaling is strictly better |
| Scheduled Scaling | Scale on cron schedule | Predictable load patterns | "traffic spikes every Monday 9am" |
| Predictive Scaling | ML forecasts future load; scales proactively | Cyclical workloads with lead time | "prevent under-provisioning before traffic spike" |

### **ASG Termination Policy**

Default: AWS terminates the instance in the AZ with the most instances, selecting the one with the oldest launch template/config, then the one closest to billing hour. Important for cost optimization.

| 💡 TIP: Lifecycle Hooks: Pause instance launch or termination to run custom scripts (install software, drain connections, backup). Uses EventBridge or SQS/SNS to trigger Lambda. |
| :---- |

## **EC2 Placement Groups**

| Strategy | Behavior | Use Case | Limitation |
| ----- | ----- | ----- | ----- |
| Cluster | All instances in same AZ, same rack — ultra-low latency | HPC, tightly coupled distributed computing, big data jobs needing low latency | Risk: single rack failure takes all instances. Cannot span AZs. |
| Spread | Each instance on distinct hardware — max 7 per AZ per group | Critical instances that must survive hardware failure (Kafka brokers, ZooKeeper) | Max 7 instances per AZ — hard limit |
| Partition | Groups of instances (partitions) on separate racks; up to 7 partitions/AZ | Distributed databases (Cassandra, HBase, Hadoop) needing rack awareness | Instances in same partition share rack risk |

| 🧠 MNEMONIC: CSP \= Cluster (Speed), Spread (Safety), Partition (Scale-Safety). Choose by priority: need speed → Cluster; need isolation per instance → Spread; need rack-aware distribution → Partition. |
| :---- |

## **EC2 Performance Networking: ENA vs. EFA**

| Feature | Elastic Network Adapter (ENA) | Elastic Fabric Adapter (EFA) |
| ----- | ----- | ----- |
| Capabilities | Provides traditional enhanced networking with high throughput (up to 100 Gbps) and high packet-per-second (PPS) performance. | Adds OS-bypass functionality to ENA, allowing applications to communicate directly with network hardware without OS kernel intervention. |
| Use Case | Standard high-throughput, network-intensive workloads. | Mandatory for tightly-coupled, highly-parallel High Performance Computing (HPC) workloads using Message Passing Interface (MPI). |

---

<br><br><br>

# **AWS Lambda**

Lambda runs code without provisioning servers. The exam tests you on limits, triggers, integrations, and when NOT to use Lambda.

## **Lambda Key Facts for the Exam**

| Property | Limit / Detail |
| ----- | ----- |
| Max execution timeout | 15 minutes (900 seconds) |
| Max memory | 128 MB to 10,240 MB (10 GB); CPU scales proportionally with memory |
| Max deployment package | 50 MB (zip), 250 MB (unzipped), 10 GB (container image) |
| Max /tmp storage | 10 GB (was 512 MB until 2023 increase) |
| Concurrent executions (default) | 1,000 per region (soft limit; can be increased) |
| Reserved concurrency | Guarantees N executions for one function; reduces pool for others |
| Provisioned concurrency | Pre-warms instances to eliminate cold starts — costs extra |
| Event sources (triggers) | S3, DynamoDB Streams, Kinesis, SQS, SNS, EventBridge, API Gateway, ALB, Cognito, etc. |
| Pricing | Per request ($0.20/million) \+ per GB-second of compute |

## **Lambda vs EC2 — When to Choose Each**

| Choose Lambda When... | Choose EC2 When... |
| ----- | ----- |
| Event-driven, sporadic traffic | Continuously running processes |
| Tasks under 15 minutes | Long-running jobs (\> 15 min) |
| No server management desired | Full OS control needed |
| Variable/unpredictable load | Predictable steady-state load (Reserved Instances save money) |
| Simple stateless functions | Stateful applications with local storage needs |

## **Lambda Destinations & Error Handling**

**Asynchronous Lambda:** Lambda retries twice on failure automatically. You can configure an on-success and on-failure destination (SQS, SNS, EventBridge, another Lambda). This is the modern replacement for Dead Letter Queues (DLQs), though DLQs still work.

| 💡 TIP: For SQS triggers: Lambda polls SQS and processes up to 10 messages per batch. On failure, messages return to the queue (up to maxReceiveCount), then go to DLQ. Use partial batch response to report only failed messages. |


<br><br><br><br><br>

# **Containers on AWS: ECS, EKS, Fargate**

Containers abstract the runtime environment. The exam asks which container platform fits the scenario.

| Service | Description | Launch Types | When to Choose |
| ----- | ----- | ----- | ----- |
| ECS (Elastic Container Service) | AWS-native container orchestration; simpler, tightly integrated | EC2 (you manage servers) or Fargate (serverless) | "AWS-native", "simpler Kubernetes alternative", "Docker on AWS" |
| EKS (Elastic Kubernetes Service) | Managed Kubernetes; industry standard; portable | EC2, Fargate, or AWS Outposts | "Kubernetes", "multi-cloud", "team knows K8s", "complex orchestration" |
| Fargate | Serverless compute engine for containers (ECS or EKS) | N/A — it IS the launch type | "no server management", "don't want to patch EC2" |
| App Runner | Fully managed; source code or container → auto-scaled web service | Fully managed (no choice) | "simplest deployment", "just run my web app", "developer-focused" |
| ECR (Elastic Container Registry) | Managed Docker container registry | N/A — storage only | Store Docker images privately on AWS; integrates natively with ECS/EKS |

| ⚠️ EXAM TRAP: Fargate is NOT a service you choose independently — it's a launch type for ECS or EKS. When a question says 'serverless containers,' they want ECS/EKS with Fargate launch type, NOT a separate Fargate service. |
| :---- |

## **EKS Pod Networking (Amazon VPC CNI)**

| Feature | Description | Key Exam Fact |
| ----- | ----- | ----- |
| VPC CNI Plugin | Native networking plugin for Amazon EKS. | Allocates real VPC private IP addresses directly to Kubernetes pods running within EKS. |
| Custom Subnet Association | Allows association of custom subnets with the EKS cluster using VPC CNI custom network configuration. | Solves IP exhaustion and meets strict security isolation requirements by assigning pod IPs from specific, secure subnets. |

| NEXT: SECTION 3 — STORAGE: S3, EBS, EFS, GLACIER & MORE |
| :---: |



# **S3 — Simple Storage Service**

S3 is AWS's most versatile storage service. The exam has MANY S3 questions spanning security, performance, durability, and costs.

## **S3 Storage Classes — Pick the Right One**

| Storage Class | Use Case | Min Duration | Min Object Size | Retrieval Time | Cost Pattern |
| ----- | ----- | ----- | ----- | ----- | ----- |
| S3 Standard | Frequently accessed data | None | None | Milliseconds | Highest per-GB storage; no retrieval fee |
| S3 Intelligent-Tiering | Unknown or changing access patterns | None | None (\< 128KB always in Frequent) | Milliseconds–minutes | Monitoring fee; auto-moves between tiers |
| S3 Standard-IA | Infrequently accessed, rapid retrieval needed | 30 days | 128 KB | Milliseconds | Lower storage; retrieval fee charged |
| S3 One Zone-IA | Infrequent access, can tolerate AZ loss | 30 days | 128 KB | Milliseconds | 20% cheaper than Standard-IA; one AZ only |
| S3 Glacier Instant Retrieval | Archives accessed once a quarter | 90 days | 128 KB | Milliseconds | Cheap storage; retrieval fee |
| S3 Glacier Flexible Retrieval | Archives; flexible retrieval time OK | 90 days | 40 KB | 1min – 12hrs | Cheaper storage; higher retrieval latency |
| S3 Glacier Deep Archive | Long-term compliance archives (7–10 yrs) | 180 days | 40 KB | 12–48 hours | Cheapest; highest retrieval latency |
| S3 Express One Zone | Ultra-low latency, high-throughput workloads | None | None | Single-digit ms | Premium; single AZ; new (2023) |

| 💡 TIP: Lifecycle Policies automate transitions. Common exam pattern: Standard → Standard-IA (after 30 days) → Glacier (after 90 days) → Deep Archive (after 365 days) → Expire. Draw this pipeline when you see 'cost optimization \+ storage' questions. |
| :---- |

| ⚠️ EXAM TRAP: S3 requires a minimum of 30 days in the current storage class before you can transition objects to S3 Standard-IA or S3 One Zone-IA. If an exam scenario suggests transitioning to Standard-IA after only 7 days, it is technically impossible and incorrect! |
| :---- |

## **S3 Security**

S3 security has multiple layers. The effective access is the UNION of all applicable policies (unless there's a Deny).

### **S3 Access Control Mechanisms**

| Mechanism | What Controls | Scope | When to Use |
| ----- | ----- | ----- | ----- |
| Bucket Policy (Resource-based) | JSON policy on bucket; can grant cross-account access | Bucket \+ Object | Public access, cross-account, IP restrictions, HTTPS-only enforcement |
| IAM Policy (Identity-based) | Policy on IAM user/role/group | Identity | Internal AWS users/roles accessing S3 |
| S3 ACLs (deprecated) | Per-object or per-bucket access lists | Object or Bucket | Legacy; prefer bucket policies. AWS recommends disabling ACLs. |
| S3 Block Public Access | Account or bucket-level override that blocks any public access settings | Account or Bucket | First line of defense; overrides bucket policies and ACLs |
| S3 Object Ownership | Controls who owns uploaded objects; can disable ACLs entirely | Bucket | When using Block Public Access; simplifies ownership |
| VPC Endpoint Policy | Controls S3 access from within VPC via Gateway Endpoint | Endpoint | Prevent S3 access from going to internet from VPC |
| S3 Object Lock (Compliance Mode) | Strictly enforces WORM; cannot be bypassed | Object version | Regulatory compliance; no user (even Root) can delete or modify settings during the retention period |
| S3 Object Lock (Governance Mode) | Enforces softer WORM | Object version | Protect from most users; can be bypassed with `s3:BypassGovernanceRetention` permission or the Root account |
| S3 Access Points | Dedicated hostnames for bucket access | Bucket \+ Prefix | Simplify access control when scaling permissions for multiple diverse teams (e.g., 20 different teams in AWS Organizations) |

| ⚠️ EXAM TRAP: Block Public Access has 4 settings (BlockPublicAcls, IgnorePublicAcls, BlockPublicPolicy, RestrictPublicBuckets). If ANY of the 'block' settings is ON at account level, it overrides bucket-level settings. This is your first safeguard against accidental public exposure. |
| :---- |

### **S3 Encryption Options**

| Encryption Type | Key Management | Who Manages Keys | Exam Keyword |
| ----- | ----- | ----- | ----- |
| SSE-S3 (AES-256) | S3 manages keys completely | AWS | "simplest", "no key management overhead", default since Jan 2023 |
| SSE-KMS | AWS KMS manages keys; audit trail in CloudTrail | AWS KMS (you control key policy) | "audit who decrypted", "compliance", "key rotation needed" |
| SSE-C | Customer provides key in each API request; S3 uses and discards it | Customer | "bring your own key", "customer managed external key"; must use HTTPS |
| Client-Side Encryption | Encrypt before upload; decrypt after download | Customer (fully) | "encrypt before sending to AWS", "data never plaintext in AWS" |
| DSSE-KMS | Dual-layer encryption (KMS) | AWS KMS | "double encryption", "government/defense compliance" |

## **S3 Replication**

| Feature | CRR (Cross-Region) | SRR (Same-Region) |
| ----- | ----- | ----- |
| Purpose | Disaster recovery, lower latency to other regions, compliance | Log aggregation, sync prod to test, data sovereignty within region |
| Versioning Required? | Yes — both source and destination | Yes — both source and destination |
| Existing Objects | Not replicated automatically — use S3 Batch Replication for existing objects | Same — Batch Replication for existing |
| Delete markers | NOT replicated by default (opt-in) | NOT replicated by default (opt-in) |
| Replication of Replicas | No — no chaining (A→B→C won't work unless you replicate A→C too) | Same limitation |

| 💡 TIP: S3 Replication Time Control (RTC): Guarantees 99.99% of objects replicated within 15 minutes. Use when you have an RPO (Recovery Point Objective) requirement of 15 minutes. |
| :---- |

## **S3 Performance**

| Technique | Detail |
| ----- | ----- |
| Multi-part Upload | Recommended for files \> 100 MB; required for \> 5 GB; enables parallel upload |
| S3 Transfer Acceleration | Uses CloudFront edge locations \+ AWS backbone to speed up uploads from distant users |
| Byte-Range Fetches | Download specific byte ranges in parallel; also recover partial files |
| S3 Select / Glacier Select | Server-side filtering using SQL expressions; reduces data transferred from S3 to application |
| S3 Object Lambda | Inserts a Lambda function into S3 GET requests to automatically transform/filter data on-the-fly (e.g., redact PII, resize images) before returning it to the calling application, eliminating the need to store derivative copies. |
| Baseline performance | 3,500 PUT/COPY/POST/DELETE per second per prefix; 5,500 GET/HEAD per second per prefix |

| ⚠️ EXAM TRAP: S3 is NOT a filesystem. You cannot mount S3 natively like NFS/SMB. Use EFS for shared filesystem needs. However, S3 File Gateway (Storage Gateway) can present S3 as an NFS/SMB mount for on-premises workloads. |
| :---- |

---

# **Storage Gateway**

Storage Gateway bridges on-premises infrastructure with AWS cloud storage. Critical for hybrid cloud exam scenarios.

| Gateway Type | Protocol | Backend Storage | Use Case |
| ----- | ----- | ----- | ----- |
| S3 File Gateway | NFS / SMB | Amazon S3 | Store files as S3 objects; on-prem apps accessing S3 via file protocol |
| FSx File Gateway | SMB | Amazon FSx for Windows | Cache frequently accessed Windows files on-prem; Windows AD integration |
| Volume Gateway (Cached) | iSCSI block | S3 (primary) \+ local cache | Keep recently accessed data on-prem; rest in S3 — low latency for hot data, minimizing local storage expansion |
| Volume Gateway (Stored) | iSCSI block | On-prem (primary) \+ S3 async backup | Keep full dataset on-prem; async backup to S3 as EBS snapshots |
| Tape Gateway | iSCSI VTL (Virtual Tape Library) | S3 / Glacier / Deep Archive | Replace physical tape with virtual tapes without changing existing backup application workflows; integrate with Veeam/Commvault |

| 🧠 MNEMONIC: File = NFS/SMB → S3 // Volume = Block (iSCSI) // Tape = VTL. If the question says on-prem NFS access to S3 → S3 File Gateway. If it says replace physical tape backup → Tape Gateway. |
| :---- |

<br>

# **Database Selection Guide**

Choosing the right database is a core SAA skill. The exam gives you a scenario and expects you to match it to the correct DB engine.

| Database | Type | Best For | Key Differentiators |
| ----- | ----- | ----- | ----- |
| RDS (MySQL, Postgres, etc.) | Relational (SQL) | Traditional OLTP, existing SQL apps | Managed; automated backups; Multi-AZ; Read Replicas |
| Aurora | Relational (MySQL/Postgres compatible) | High-performance OLTP, SaaS | 5x faster than MySQL; 6-copy storage across 3 AZs; Aurora Serverless |
| DynamoDB | NoSQL Key-Value \+ Document | High-scale internet apps, gaming, IoT, session data | Single-digit ms at any scale; serverless; global tables |
| ElastiCache (Redis) | In-Memory Cache | Session store, real-time leaderboards, pub/sub, ML feature store | Sub-ms latency; persistence options; cluster mode |
| ElastiCache (Memcached) | In-Memory Cache (simpler) | Simple distributed cache; stateless | No persistence; no replication; multi-threaded |
| Redshift | Columnar Data Warehouse (SQL) | OLAP, BI, analytics on large datasets | Columnar storage; MPP; Redshift Spectrum queries S3 |
| Neptune | Graph | Social networks, fraud detection, knowledge graphs | Gremlin (property graph) \+ SPARQL (RDF) |
| DocumentDB | Document (MongoDB-compatible) | Content management, catalogs, user profiles | MongoDB API compatibility; managed; 6 copies like Aurora |
| Keyspaces | Wide Column (Cassandra-compatible) | IoT, time-series, high-write workloads | Serverless; CQL compatible; pay per request |
| Timestream | Time Series | IoT sensor data, DevOps metrics, telemetry | Built-in time-series analytics; auto-tiering |
| QLDB | Ledger (immutable) | Financial transactions, supply chain audit trail | Cryptographically verifiable; centrally owned |
| OpenSearch Service | Search \+ Analytics | Log analytics, full-text search, dashboards | Elasticsearch API compatible; pairs with Kibana/OpenSearch Dashboards |









# **RDS — Relational Database Service**

## **RDS Multi-AZ vs Read Replicas — The Classic Confusion**

| Feature | Multi-AZ | Read Replica |
| ----- | ----- | ----- |
| Purpose | HIGH AVAILABILITY (failover) | PERFORMANCE (read scaling) |
| Replication | Synchronous — standby always in sync | Asynchronous — slight lag possible |
| Readable? | NO — standby is passive; only used during failover | YES — serve read traffic |
| Failover time | \~1-2 minutes (automatic DNS flip) | Must manually promote to primary (not automatic) |
| Cross-region? | Only within same region | Can be cross-region (for disaster recovery) |
| Cross-AZ cost? | Free data transfer for Multi-AZ replication | Cross-region replica has data transfer cost |
| Exam keyword | "availability", "failover", "disaster recovery" | "performance", "read traffic", "reporting", "analytics" |

| ⚠️ EXAM TRAP: CRITICAL: Multi-AZ does NOT improve read performance — the standby is not readable. If a question says 'improve read performance AND high availability,' the answer uses BOTH Multi-AZ AND Read Replicas. |
| :---- |

## **RDS Backup & Restore**

| Feature | Details |
| ----- | ----- |
| Automated Backups | Daily snapshot \+ transaction logs → point-in-time restore (PITR) up to 35 days. Stored in S3. |
| Manual Snapshots | User-initiated; retained indefinitely (until you delete). Shared across regions/accounts. |
| Restore behavior | Always creates a NEW DB instance — you cannot restore in-place. Update app connection string. |
| RDS Backup window | Configure during low-traffic period. I/O may be briefly suspended for single-AZ. |
| Deletion protection | Enable to prevent accidental deletion. Must disable before terminating. |


## **RDS Proxy**

| Feature | Detail |
| ----- | ----- |
| Connection Pooling | Sits between serverless applications (e.g., AWS Lambda) and the database to pool and share connections, preventing database connection limits from being exhausted during rapid scaling. |
| Failover Benefits | Reduces database failover times for Amazon RDS Multi-AZ by up to 66% and handles failovers transparently without requiring application code adjustments. |

---

# **Aurora — The Exam Favorite**

Aurora is AWS's own cloud-native relational database. It appears frequently because of its unique architecture.

## **Aurora Architecture Highlights**

| Feature | Aurora | Standard RDS |
| ----- | ----- | ----- |
| Storage | Shared cluster volume: 6 copies across 3 AZs, auto-grows 10 GB increments to 128 TB | Single instance storage or gp3/io1 EBS |
| Read Replicas | Up to 15 Aurora Replicas (vs 5 for RDS) | Up to 5 read replicas |
| Failover | < 30 seconds (usually < 10 sec) — promotes replica automatically | ~1-2 minutes for Multi-AZ |
| Backtrack | Rewind database to a point in time WITHOUT restore (seconds, up to 72 hrs back) | Not available — must restore from backup to new instance |
| Aurora Serverless v2 | Auto-scales read/write capacity in fractions of ACUs — pay per use | Must choose instance size upfront |
| Global Database | 1 primary region + up to 5 secondary read regions; < 1 sec replication lag; RTO < 1 min | Cross-region read replicas (higher lag) |
| Aurora Auto Scaling | Auto-scales read replicas automatically based on CPU utilization or average connections | Requires manual or Application Auto Scaling setup |
| Fast Database Cloning | Instantaneous copy-on-write clone sharing underlying storage volume; zero storage cost until changes are written | Snapshot restore creates a brand-new instance (slower, full storage cost) |

| 💡 TIP: Aurora Global Database for exam: If the question says 'RPO of seconds' or 'RTO under 1 minute' across regions, Aurora Global Database is the answer. It can also be used for local reads to reduce latency globally. |
| :---- |

<br>


# **DynamoDB — The Serverless NoSQL Giant**

DynamoDB is deeply integrated across AWS. Know its consistency models, capacity modes, indexing, and streams cold.

## **DynamoDB Core Concepts**

| Concept | Detail | Exam Implication |
| ----- | ----- | ----- |
| Primary Key | Can be simple (Partition Key only) or composite (Partition Key \+ Sort Key) | Design partition key for uniform distribution — avoid hot partitions |
| Read Consistency | Eventually Consistent (default, cheaper, 0.5 RCU per 4KB) or Strongly Consistent (1 RCU per 4KB) | Financial apps needing latest data → Strongly Consistent; extra 2x RCU cost |
| Capacity Modes | Provisioned (specify RCU/WCU; predictable cost) vs On-Demand (pay per request; variable load) | On-Demand for unpredictable; Provisioned \+ Auto Scaling for steady with spikes |
| DynamoDB Streams | Ordered stream of item changes (24-hour retention); triggers Lambda, Kinesis | Event-driven patterns: change data capture, cross-region replication, triggers |
| TTL (Time to Live) | Automatically delete items after timestamp expires; no cost for deletions | Session expiry, log retention, ephemeral data |
| DynamoDB Accelerator (DAX) | Write-through in-memory cache; microsecond read latency; no app code changes for reads | DAX is for read caching. DAX caches individual item reads and query results. |
| Global Tables | Multi-region, multi-active replication; DynamoDB Streams must be enabled | Active-active replication: write to any region, read from nearest |
| Transactions | ACID transactions across multiple items/tables using TransactWrite and TransactGet | Financial apps, order management, inventory systems |

## **DynamoDB Indexes**

| Index Type | Created When | Key Structure | Read Consistency | Throughput | Use Case |
| ----- | ----- | ----- | ----- | ----- | ----- |
| LSI (Local Secondary Index) | Table creation ONLY | Same partition key, different sort key | Strongly consistent OK | Shares table's provisioned capacity | Query same PK with different sort order |
| GSI (Global Secondary Index) | Any time | Different partition and/or sort key | Eventually consistent ONLY | Has its own provisioned capacity | Query on any attribute; most flexible |

| 🧠 MNEMONIC: LSI \= Local \= Same partition key \= created at table Launch. GSI \= Global \= Greater flexibility \= can be added anytime, own capacity. |
| :---- |

<br>

# **ElastiCache — Caching Strategies**

| Strategy | How it Works | Best For | Risk |
| ----- | ----- | ----- | ----- |
| Lazy Loading (Cache-Aside) | Check cache → miss → fetch from DB → write to cache | Read-heavy, tolerance for stale data | Cache miss penalty (3 trips); stale data if DB changes without cache update |
| Write-Through | Write to DB and cache simultaneously | Data freshness critical; small dataset | Write penalty (2 trips per write); cache churn if rarely read |
| Write-Behind (Write-Back) | Write to cache first; async write to DB | Write-heavy workloads | Data loss if cache fails before DB write |
| Cache TTL | Items expire after N seconds | Prevent stale data buildup | Too short \= cache misses; too long \= stale data |

| ⚠️ EXAM TRAP: Redis vs Memcached for the exam: If the question mentions ANY of these → choose Redis: persistence, pub/sub, Lua scripting, sorted sets (leaderboards), geospatial, replication, Multi-AZ, backups, or complex data structures. Memcached is only chosen for 'simple, stateless cache needing horizontal scale with multiple CPU cores.' |
| :---- |

## **ElastiCache Security (Redis AUTH)**

| Feature | Detail |
| ----- | ----- |
| Redis AUTH | Requires clients to enter a token (password) before executing commands, protecting sensitive cache data (e.g., banking portals). |
| In-Transit Encryption | Must be explicitly enabled on the ElastiCache cluster before Redis AUTH can be configured. |

| NEXT: SECTION 5 — NETWORKING: VPC, ROUTE 53, CLOUDFRONT & MORE |
| :---- |

<br>



# **VPC — Virtual Private Cloud**

VPC is your private network in AWS. The exam tests VPC design constantly — subnets, routing, security, and connectivity patterns.

## **VPC Fundamentals**

| Component | Description | Key Facts |
| ----- | ----- | ----- |
| VPC | Logically isolated network; spans all AZs in a region | Max 5 VPCs per region (soft limit); CIDR range /16 to /28 |
| Subnet | AZ-specific subdivision of a VPC CIDR block | Public (route to IGW) vs Private (no IGW route). AWS reserves 5 IPs per subnet (first 4 + last). |
| Internet Gateway (IGW) | Enables internet access for public subnets; horizontally scaled, HA, no bandwidth limit | One IGW per VPC; attach to VPC, add route in public subnet RT |
| NAT Gateway | Allows PRIVATE subnet instances to initiate outbound internet connections; blocks inbound | Regional HA option (not AZ-resilient); deployed IN public subnet; uses Elastic IP |
| Route Table | Rules directing traffic based on destination CIDR | Each subnet associates with one RT; subnets without explicit association use main RT |
| Security Group (SG) | Stateful virtual firewall at instance level; only ALLOW rules | Stateful = return traffic auto-allowed. Default: all outbound allowed, all inbound denied. |
| Network ACL (NACL) | Stateless firewall at subnet level; ALLOW and DENY rules | Stateless = must explicitly allow return traffic (ephemeral ports 1024-65535). Evaluated in order by rule number. |
| VPC Peering | Direct private connection between two VPCs (same or different account/region) | NOT transitive — A↔B and B↔C does NOT mean A↔C. No overlapping CIDRs allowed. |
| VPC Endpoints | Private connection to AWS services without internet | **Gateway**: S3 & DynamoDB only; uses prefix lists in route tables; completely free. **Interface**: Powered by PrivateLink; uses ENI with private IP from subnet CIDR; requires DNS entries; incurs hourly + data charges. |
| VPC Endpoint Policies | Controls access via endpoints | You can restrict S3 bucket access to strictly allow requests from a specific Gateway Endpoint using the `aws:sourceVpce` condition key in the bucket policy. |

| ⚠️ EXAM TRAP: NAT Gateway is NOT highly available across AZs by default. For true HA, deploy one NAT Gateway per AZ and configure each private subnet to route to the NAT Gateway in ITS OWN AZ. |
| :---- |

## **Security Group vs NACL — The Classic Comparison**

| Property | Security Group | Network ACL (NACL) |
| ----- | ----- | ----- |
| Level | Instance (ENI) | Subnet |
| State | STATEFUL — return traffic auto-allowed | STATELESS — must allow both directions |
| Rules | Allow only (whitelist) | Allow AND Deny |
| Evaluation | All rules evaluated together | Rules evaluated in numerical order (lowest first); first match wins |
| Default behavior | Deny all inbound; allow all outbound | Allow all traffic (default VPC NACL) |
| Use case | Primary security control per workload | Additional subnet-level security, blocking specific IPs (DDoS) |

| 💡 TIP: Security Groups can reference other Security Groups by ID (not CIDR). This is powerful: allow inbound from 'sg-webserver' to 'sg-database' without knowing IP addresses. This is the correct pattern for tiered application security. |
| :---- |

## **VPC Connectivity Options**

| Option | Use Case | Key Characteristics |
| ----- | ----- | ----- |
| VPC Peering | Connect 2 VPCs directly | Non-transitive; 1:1 relationship; any account/region; no overlapping CIDR |
| Transit Gateway (TGW) | Hub-and-spoke: connect many VPCs + on-prem | Transitive routing; 1 TGW can handle thousands of VPCs; route tables on TGW; share via RAM |
| AWS PrivateLink | Expose service to other VPCs without peering | Secure cross-account API sharing without internet or VPN. Provider VPC must front the service with an NLB; consumer VPC connects using an Interface Endpoint. Unidirectional; doesn't expose full VPC. |
| Site-to-Site VPN | Encrypted IPSec tunnel over public internet | Fast to set up (~hours); variable latency; up to 1.25 Gbps per tunnel |
| Direct Connect (DX) | Dedicated private line from on-prem to AWS | Consistent low latency; 1/10/100 Gbps; no encryption by default; long setup (weeks/months) |
| Direct Connect + VPN | Encrypted tunnel over Direct Connect | Best of both: dedicated bandwidth + encryption for compliance |
| VPN CloudHub | Hub-and-spoke VPN for multiple on-prem sites | Uses Virtual Private Gateway; sites can communicate through AWS |

| 🧠 MNEMONIC: VPN = Variable/Virtual (goes over internet, encrypted, variable latency). DX = Dedicated eXpress (physical line, consistent, no encryption). Need both consistency AND encryption? DX + VPN. |
| :---- |

### **DNS Hostnames and Resolution in Custom VPCs**

| Configuration Parameter | Default Behavior | Exam Implication |
| ----- | ----- | ----- |
| **`enableDnsSupport`** | Enabled by default in custom VPCs | Enables DNS resolution via the Amazon-provided DNS server. |
| **`enableDnsHostnames`** | Disabled by default in custom VPCs | Must be explicitly enabled alongside `enableDnsSupport` to ensure EC2 instances launched with public IPs automatically receive a public DNS hostname. |

## **VPC Flow Logs**

**What it captures:** IP traffic metadata (not packet contents) for VPC, subnet, or ENI. Goes to CloudWatch Logs, S3, or Kinesis Data Firehose.

**Use cases:** Troubleshoot connectivity issues, security analysis, compliance auditing. Flow logs do NOT capture: DNS traffic, Windows license activation, 169.254.x.x (metadata), DHCP traffic.

---


---

# **Route 53 — DNS & Routing Policies**

Route 53 is AWS's DNS service. The SAA exam tests routing policies extensively — know which policy to pick in each scenario.

## **Route 53 Routing Policies**

| Policy | How it Works | Use Case | Exam Scenario Keyword |
| ----- | ----- | ----- | ----- |
| Simple | Maps domain to one or more resources; random selection if multiple values | Single resource or stateless one-backend | "basic DNS", "single endpoint" |
| Weighted | Split traffic by weights (0-255); weight 0 \= no traffic | A/B testing, blue/green deployments, gradual migration | "split 10% to new version", "blue/green", "canary" |
| Latency-based | Routes to region with lowest network latency for user | Multi-region app, global users wanting fast response | "lowest latency", "fastest response for users globally" |
| Failover | Active-Passive; routes to secondary when primary health check fails | Disaster recovery, active-standby architecture | "failover", "DR", "primary fails → secondary" |
| Geolocation | Routes based on user's geographic location (continent, country, state) | Content localization, GDPR data residency, language routing | "users in Europe to EU servers", "data residency", "localization" |
| Geoproximity | Routes based on location \+ optional bias to shift traffic toward/away from a region | Fine-grained geographic traffic shifts; expand/shrink region coverage | "shift more traffic to new region", "traffic engineering by geography" |
| Multi-Value Answer | Returns up to 8 healthy records (like Simple \+ health checks) | Client-side load balancing across multiple endpoints | "simple load balancing", "up to 8 endpoints", "NOT a replacement for ELB" |
| IP-Based | Routes based on client IP CIDR blocks you define | Route ISP traffic to specific endpoints, cost optimization | "route specific IP ranges", "ISP-based routing" |

| ⚠️ EXAM TRAP: Geolocation vs Geoproximity: Geolocation is STRICT (users from France go to EU endpoint — full stop). Geoproximity uses location \+ BIAS to expand or shrink a region's effective coverage area — it's about TRAFFIC ENGINEERING, not strict regional routing. |
| :---- |

## **Route 53 Health Checks**

| Type | What it Checks |
| ----- | ----- |
| Endpoint Health Check | HTTP/HTTPS/TCP check to a specific IP or hostname; 15 global checkers; 3/5 threshold for pass/fail |
| Calculated Health Check | Aggregate multiple health checks with AND/OR logic; parent health based on child checks |
| CloudWatch Alarm | Health based on CW metric/alarm state — use this for private resources (no internet access) |

---


---

# **CloudFront — Content Delivery Network**

CloudFront is AWS's global CDN. It caches content at 400+ edge locations worldwide and integrates deeply with AWS services.

## **CloudFront Key Concepts**

| Concept | Detail | Exam Implication |
| ----- | ----- | ----- |
| Origin | Where CloudFront fetches content from: S3, ALB, EC2, API Gateway, any HTTP server | S3 origin: use Origin Access Control (OAC) not OAI (legacy). ALB/EC2: must be publicly accessible or use VPC Origin. |
| Distribution | A CloudFront deployment with one or more origins and behaviors | One distribution can have multiple origins with different path patterns (e.g., /api/\* → ALB, /\* → S3) |
| Edge Location | 400+ global PoPs that cache content | Not the same as a Region or AZ |
| Cache Behaviors | Rules for how CloudFront caches and routes requests based on path patterns | Order matters — more specific paths should be listed first |
| TTL | Cache-Control and Expires headers control TTL; override in CloudFront behavior | Minimum TTL, Default TTL, Maximum TTL — CF uses min of max TTL and header value |
| OAC (Origin Access Control) | Grants CloudFront permission to access private S3 bucket; users cannot access S3 directly | Replace all OAI with OAC; prevents direct S3 URL access bypassing CloudFront |
| Signed URLs / Signed Cookies | Restrict content to authenticated users; signed with trusted key pair | Signed URL: one specific file. Signed Cookies: multiple files / membership access. |
| WAF Integration | AWS WAF on CloudFront protects globally at edge | Stops attacks before they reach origin; geo-blocking, rate limiting, SQL injection protection |
| Lambda@Edge / CloudFront Functions | Run code at edge: Lambda@Edge at regional edge (heavier), CF Functions at edge location (lighter) | Auth, A/B testing, URL rewrites. CF Functions: \< 1ms, \< 2MB. Lambda@Edge: up to 30s, 10GB RAM. |
| Field-Level Encryption | Encrypt specific POST fields (e.g. credit card) at edge; only specific origin servers can decrypt | PCI compliance, protect sensitive fields in transit beyond HTTPS |

| 💡 TIP: CloudFront vs S3 Transfer Acceleration: Both speed up S3 uploads/downloads using AWS edge network. CloudFront \= CDN for content delivery (reads primarily). S3 Transfer Acceleration \= upload acceleration via AWS backbone, specifically for S3 PUT operations from distant clients. |
| :---- |

## **CloudFront Price Classes**

| Price Class | Edge Locations Included | Cost-Optimization Strategy |
| ----- | ----- | ----- |
| **Price Class 100** | US, Canada, Europe | Most cost-effective. Choose this if your customers are concentrated only in these regions (e.g., U.S. and Canada) to lower bills without paying for unused worldwide edges. |
| **Price Class 200** | Adds Asia, Middle East, Africa | Use for a balance of global reach and cost control. |
| **Price Class All** | All global Edge Locations | Best global performance, but carries the highest cost. |


<br><br><br><br>

# **Elastic Load Balancing — Three Types Compared**

Knowing WHICH load balancer to pick is a key SAA skill. The exam describes a requirement and expects you to match it.

| Feature | ALB (Application) | NLB (Network) | GWLB (Gateway) |
| ----- | ----- | ----- | ----- |
| OSI Layer | Layer 7 (HTTP/HTTPS/WebSocket/gRPC) | Layer 4 (TCP/UDP/TLS) | Layer 3+4 (IP packets) |
| Protocol | HTTP, HTTPS, WebSocket, gRPC | TCP, UDP, TLS | GENEVE (6081) for packet encapsulation |
| Performance | High throughput; some latency | Extreme performance; millions of req/sec; ultra-low latency | Transparent to traffic; used for appliances |
| Health checks | HTTP/HTTPS-based | TCP, HTTP, HTTPS | TCP, HTTP, HTTPS |
| Target types | EC2, IP, Lambda, containers | EC2, IP, ALB (as target) | EC2 instances running security appliances |
| Sticky sessions | Yes (application-based or duration-based cookies) | Yes (source IP) | No |
| WebSocket | Native support | Yes (Layer 4 pass-through) | N/A |
| Fixed IP | No (use NLB for fixed IP) | Yes (Elastic IP per AZ) | No |
| Use case | Web apps, microservices, content routing | TCP apps, gaming, IoT, NLB → ALB pattern | Deploy 3rd-party virtual appliances (Palo Alto, Fortinet) |
| Exam keyword | "HTTP routing", "host/path-based routing", "microservices", "Lambda" | "static IP", "TCP", "extreme performance", "UDP", "gaming" | "intrusion detection", "firewall appliance", "security inspection" |

| ⚠️ EXAM TRAP: NLB provides static IP addresses (one per AZ) — use this when clients need to whitelist fixed IPs. ALB has no fixed IPs. A common pattern: NLB in front of ALB to get fixed IPs while retaining Layer 7 routing. |
| :---- |

## **ALB Advanced Routing Features**

| Feature | Description | Example |
| ----- | ----- | ----- |
| Host-based routing | Route based on HTTP Host header | api.example.com → API Target Group; app.example.com → App Target Group |
| Path-based routing | Route based on URL path | /images/\* → Image servers; /api/\* → API servers |
| HTTP header routing | Route based on any HTTP header value | User-Agent: iPhone → mobile backend |
| HTTP method routing | Route based on GET/POST/PUT etc. | POST /orders → Order service TG |
| Query string routing | Route based on query string parameters | ?platform=mobile → Mobile TG |
| Fixed response | Return HTTP 200/301/302 directly from ALB | Maintenance page, redirects |
| Redirect action | ALB redirects HTTP → HTTPS | Enforce HTTPS without backend code |
| Lambda target | ALB invokes Lambda directly as target | Serverless web apps behind ALB |

## **Connection Draining / Deregistration Delay**

**What it is:** When an instance is deregistered or marked unhealthy, existing connections are kept alive for up to 3600 seconds (default 300 seconds) to complete in-flight requests before the instance is removed.

**Exam tip:** If the question says 'long-running uploads are being interrupted during deployments' → increase deregistration delay. If it says 'deployments are slow because instances stay in draining state too long' → decrease the delay.

| NEXT: SECTION 7 — MESSAGING: SQS, SNS, EVENTBRIDGE, KINESIS & MORE |
| :---: |


# **Messaging & Event-Driven Architecture**

Decoupling components is a core AWS architectural principle. These services are the glue between components.

| Service | Pattern | Key Traits | Exam Scenario |
| ----- | ----- | ----- | ----- |
| SQS | Queue (Point-to-Point pull) | Pull model; messages stored up to 14 days; at-least-once delivery (Standard) or exactly-once (FIFO) | "decouple", "buffer requests", "worker pool", "async processing" |
| SNS | Pub/Sub (Push fan-out) | Push model; up to 12.5M subscribers per topic; no persistence; many protocols (SQS, Lambda, HTTP, email) | "fan-out", "notify multiple services", "event broadcasting" |
| EventBridge | Event bus (Pub/Sub \+ Rules) | Event routing with filter rules; integrates 200+ AWS services \+ SaaS apps; schema registry | "event-driven", "route events from AWS services", "SaaS integration" |
| Kinesis Data Streams | Real-time streaming | Retain 24hrs–365 days; multiple consumers; ordered within shard; replay capability | "real-time analytics", "clickstream", "IoT telemetry", "replay events" |
| Kinesis Data Firehose | Streaming ETL delivery | Fully managed delivery to S3/Redshift/OpenSearch; near-real-time (buffer 60s or 5MB) | "load streaming data to S3/Redshift", "transform \+ deliver", "near-real-time" |
| Kinesis Data Analytics | Real-time SQL on streams | SQL or Apache Flink on Kinesis/MSK streams; real-time aggregations/anomaly detection | "real-time SQL on streaming data", "running averages", "anomaly detection" |
| MSK (Managed Kafka) | Managed Apache Kafka | AWS manages Kafka brokers; Kafka API compatible; persistent storage on EBS | "Kafka", "existing Kafka workloads", "exactly-once semantics", "long retention" |
| MQ (Amazon MQ) | Managed message broker | ActiveMQ / RabbitMQ; for migrating on-prem apps using JMS/AMQP/MQTT/STOMP | "migrate from on-prem ActiveMQ/RabbitMQ", "MQTT", "STOMP", "AMQP" |

## **SQS Deep Dive**

| Property | Standard SQS | FIFO SQS |
| ----- | ----- | ----- |
| Ordering | Best-effort (not guaranteed) | Strict First-In-First-Out per Message Group ID |
| Delivery | At-least-once (duplicates possible) | Exactly-once processing (deduplication) |
| Throughput | Nearly unlimited | 300 msg/sec (3,000 with batching) |
| Deduplication | Not supported | 5-minute deduplication window (content or ID based) |
| Use case | High throughput, order doesn't matter | Order processing, financial transactions |
| Queue name | Any name | Must end in .fifo |

### **SQS Important Properties**

| Property | Default | Max | Purpose |
| ----- | ----- | ----- | ----- |
| Visibility Timeout | 30 seconds | 12 hours | Duration message is hidden after consumer receives it; if not deleted, becomes visible again |
| Message Retention | 4 days | 14 days | How long messages stay in queue |
| Message Size | 256 KB | 256 KB (use Extended Client Library for up to 2GB via S3) | Store large payloads in S3; SQS holds reference |
| Long Polling | 0 (short poll) | 20 seconds | Wait for messages instead of returning empty; reduces API calls and cost |
| Dead Letter Queue | Disabled | — | Messages that fail maxReceiveCount times sent to DLQ for analysis |
| Delay Queue | 0 seconds | 15 minutes | Delay delivery of new messages by N seconds; useful for dependent processes |

| 💡 TIP: Visibility Timeout confusion: If consumer crashes processing a message, the visibility timeout expires and the message becomes visible AGAIN. Set visibility timeout longer than your maximum processing time to prevent duplicate processing. |
| :---- |

## **SNS vs SQS vs EventBridge — When to Choose**

| If the scenario says... | Choose |
| ----- | ----- |
| "Fan-out to multiple services simultaneously" (e.g., order placed → notify inventory, billing, shipping) | SNS → multiple SQS queues (SNS-SQS fan-out pattern) |
| "Decouple producer from consumer; buffer traffic spikes" | SQS (with or without SNS in front) |
| "Route events from 200+ AWS services or third-party SaaS apps with filter rules" | EventBridge Event Bus |
| "Real-time stream processing, replay events, multiple consumers" | Kinesis Data Streams |
| "Deliver streaming data to S3/Redshift without managing consumers" | Kinesis Data Firehose |
| "Migrate existing RabbitMQ/ActiveMQ to managed service" | Amazon MQ |
| "Kafka workload, need Kafka API compatibility" | MSK (Managed Streaming for Kafka) |

## 

## **Kinesis Shards & Scaling**

**Shard capacity:** 1 MB/s write, 2 MB/s read per shard. Scale by adding shards (Shard Split) or combining them (Shard Merge).

**Enhanced Fan-Out:** Each consumer gets its own 2 MB/s pipe (instead of sharing 2 MB/s across all consumers). Use when you have multiple Lambda functions or applications reading the same stream and need full throughput per consumer.

| 🧠 MNEMONIC: Kinesis \= Real-time \+ Replay \+ Multiple consumers. Firehose \= Load \+ Near-real-time \+ Delivery to S3/Redshift/OpenSearch. Analytics \= SQL on streams. Streams needs producers/consumers managed; Firehose is fully managed ingestion. |
| :---- |

| NEXT: SECTION 8 — MONITORING, LOGGING, SECURITY & COMPLIANCE |
| :---: |

---

---

# **Monitoring Stack — CloudWatch, CloudTrail, Config**

| Service | What it monitors | What it stores | Exam Trigger Words |
| ----- | ----- | ----- | ----- |
| CloudWatch Metrics | Performance metrics (CPU, network, custom) | 15 months by default; detailed monitoring = 1-min granularity | "monitor CPU", "set alarm", "auto scaling trigger" |
| CloudWatch Logs | Log ingestion from EC2, Lambda, RDS, VPC Flow Logs, etc. | Retain per log group (1 day to 10 years) | "centralize logs", "filter log events", "log insights query" |
| CloudWatch Alarms | Triggered when metric crosses threshold | Actions: SNS, Auto Scaling, EC2 action | "notify when CPU > 80%", "scale when latency > 500ms" |
| CloudWatch Events / EventBridge | React to AWS API events (cron schedule or event pattern) | Routes to targets (Lambda, SQS, etc.) | "trigger Lambda on schedule", "react to EC2 state change" |
| CloudTrail (Management Events) | AWS API calls (control plane operations); 90-day free | 90-day event history free; S3 for long-term | "audit API calls", "who deleted S3 bucket", "compliance trail" |
| CloudTrail Data Events | Bucket-level object read/write operations (e.g., GetObject, PutObject) | Must be explicitly enabled; incurs additional charges | "audit S3 object access", "log PutObject calls" |
| CloudTrail Log Validation | S3 trail log integrity (tamper detection) | SHA-256 hashing and digital signatures | "cryptographic proof", "log tampering", "ensure log integrity" |
| AWS Config | Resource configuration changes + compliance rules | Configuration timeline; non-compliant alerts (e.g., unencrypted buckets, bad passwords) | "are resources compliant", "config drift", "remediation" |
| X-Ray | Distributed tracing for applications | Service maps, latency analysis, error analysis | "trace requests across microservices", "find bottleneck", "latency root cause" |
| AWS Health Dashboard | AWS service health events affecting your account | Alerts for maintenance, degradation, etc. | "notified when AWS maintenance affects my instances" |

| ⚠️ EXAM TRAP: CloudTrail vs CloudWatch: CloudTrail = WHO called what API (audit). CloudWatch = HOW is the system performing (metrics/logs). Config = IS the resource compliant with rules (configuration state). These are three different questions. |
| :---- |


# **Security Services — Know Your Defense Layers**

| Service | What it Does | Layer | Exam Scenario |
| ----- | ----- | ----- | ----- |
| AWS WAF | Web Application Firewall; rules against SQL injection, XSS, bad bots, geo-blocks | Layer 7; CloudFront, ALB, API GW, AppSync | "block SQL injection", "protect against OWASP Top 10", "geo-block countries" |
| AWS Shield Standard | Automatic DDoS protection (Layers 3/4) for all AWS customers at no cost | Layer 3/4; all resources | Always on; no configuration needed |
| AWS Shield Advanced | $3,000/month; Enhanced DDoS protection \+ 24/7 DDoS Response Team \+ cost protection | Layer 3/4/7; CloudFront, Route53, ALB, NLB, EIP | "sophisticated DDoS attacks", "financial protection", "DRT support" |
| AWS Firewall Manager | Centrally deploy and manage WAF rules, Shield Advanced, SGs, Network Firewall across org | Multi-account, Organizations-level | "apply WAF rules across all accounts", "organization-wide security policies" |
| Amazon GuardDuty | Threat detection using ML on CloudTrail, VPC Flow Logs, DNS Logs, S3 logs, EKS logs | Account-level threat intelligence | "detect cryptomining", "suspicious API calls", "unauthorized access" |
| Amazon Inspector | Automated vulnerability assessment for EC2, ECR images, Lambda functions | Workload-level CVEs | "scan EC2 for vulnerabilities", "container image scanning", "CVE detection" |
| Amazon Macie | ML-based PII/sensitive data discovery and classification in S3 | Data classification | "find PII in S3", "GDPR compliance", "sensitive data discovery" |
| AWS Security Hub | Aggregates findings from GuardDuty, Inspector, Macie, Config into unified dashboard | Cross-service aggregation | "single pane of glass for security", "normalize findings" |
| AWS Network Firewall | Managed stateful firewall (Suricata rules) for VPC — deep packet inspection | VPC level | "deep packet inspection", "IDS/IPS in VPC", "custom domain filtering" |
| Secrets Manager | Store, rotate, retrieve secrets (DB passwords, API keys); auto-rotation with Lambda | Application secrets | "rotate DB credentials automatically", "never hardcode passwords" |
| Systems Manager Parameter Store | Store config data and secrets (standard: free, advanced: $0.05/param/month) | Config & secrets | "store configuration", "free secrets storage", "hierarchy config" |
| KMS (Key Management Service) | Create and manage encryption keys; integrate with 100+ AWS services | Encryption key management | "customer managed keys", "key rotation", "envelope encryption" |
| CloudHSM | Dedicated Hardware Security Module; you manage keys entirely; FIPS 140-2 Level 3 | Hardware key protection | "FIPS 140-2 Level 3", "full key control", "regulatory compliance (PKCS11, JCE)" |

| ⚠️ EXAM TRAP: Secrets Manager vs Parameter Store: Secrets Manager costs \~$0.40/secret/month but auto-rotates credentials and is designed for secrets. Parameter Store is free for standard parameters but no auto-rotation (need to trigger Lambda yourself). For exam: 'auto-rotate database credentials' → Secrets Manager. |
| :---- |

| 🧠 MNEMONIC: GuardDuty \= Guard dog (threat DETECTION). Inspector \= Home inspector (vulnerability SCANNING). Macie \= Mac & Privacy (PII/data CLASSIFICATION). These are three different jobs: detect threats, find vulnerabilities, classify data. |
| :---- |

<br><br><br><br>


# **Core Architectural Patterns for SAA**

## **1\. Highly Available Web Application (The Classic Pattern)**

This pattern appears in some form in nearly every SAA exam. Know it cold.

| Layer | Service(s) | Why |
| ----- | ----- | ----- |
| DNS | Route 53 (Latency or Failover policy) | Global DNS with health checks; route users to nearest/healthy region |
| CDN | CloudFront | Cache static assets; reduce origin load; SSL termination at edge |
| Load Balancer | ALB (Multi-AZ) | Distribute traffic across AZs; path-based routing; health checks |
| Web Tier | EC2 in ASG across 2+ AZs (or ECS/Fargate) | Horizontal scaling; AZ fault tolerance |
| App Tier | EC2/Lambda in private subnets across 2+ AZs | Isolation from web tier; auto-scaling |
| Cache | ElastiCache (Redis) — replicated across AZs | Reduce DB read load; session storage |
| Database | Aurora Multi-AZ or RDS Multi-AZ \+ Read Replicas | HA for writes (Multi-AZ) \+ scale reads (replicas) |
| Storage | S3 for static assets and uploads | Durable, scalable object storage outside EC2 |
| Secrets | Secrets Manager or Parameter Store | No hardcoded credentials anywhere |
| Network | VPC with public/private subnets; NAT Gateway per AZ | Security isolation; outbound internet for private tier |

## **2\. Serverless Architecture Pattern**

| Layer | Service | Notes |
| ----- | ----- | ----- |
| API Layer | API Gateway (REST or HTTP API) | HTTP API \= cheaper, lower latency. REST API \= more features (usage plans, API keys, WAF). |
| Compute | Lambda | Functions triggered by API GW; 15 min max; stateless |
| Database | DynamoDB (serverless) | On-Demand capacity; single-digit ms; no server management |
| Cache | DAX (for DynamoDB) or ElastiCache | DAX \= transparent, microsecond reads. ElastiCache \= more control, broader use. |
| Auth | Cognito User Pools \+ Identity Pools | User Pool \= auth/login. Identity Pool \= exchange token for AWS credentials. |
| Storage | S3 | Store files; trigger Lambda on upload |
| Events | EventBridge or SNS \+ SQS | Async workflows between Lambda functions |
| CDN | CloudFront | Serve SPA (React/Vue) from S3 \+ API calls via CloudFront |

| 💡 TIP: REST API vs HTTP API in API Gateway: HTTP API costs 71% less and has lower latency. REST API adds: usage plans, API keys, request transformation, AWS WAF, resource policies, edge-optimized. For exam: need usage plans/throttling per client or WAF → REST API. Otherwise → HTTP API. |
| :---- |

## **3\. Disaster Recovery Patterns — RTO/RPO Decision**

| Strategy | RTO | RPO | Cost | How it Works |
| ----- | ----- | ----- | ----- | ----- |
| Backup & Restore | Hours | Hours | Lowest | Backups to S3 or Glacier; restore from scratch. No standby environment. |
| Pilot Light | 10s of minutes | Minutes | Low | Minimal core services always running (DB replication ON); scale up app tier during DR. |
| Warm Standby | Minutes | Seconds to minutes | Medium | Scaled-down full environment running; scale to production size during DR. |
| Multi-Site Active/Active | Near zero | Near zero | Highest | Full production in 2+ regions simultaneously; Route 53 routes to both. |

| 🧠 MNEMONIC: BR-PP-WS-AA: Backup-Restore (hours), Pilot-light (minutes), Warm-Standby (minutes, more ready), Active-Active (near zero). Cost goes UP as RTO/RPO goes DOWN. |
| :---- |

## **4\. Data Lake Architecture Pattern**

| Layer | Services | Purpose |
| ----- | ----- | ----- |
| Ingestion | Kinesis Data Streams / Firehose, AWS DMS, AWS DataSync, Snowball Edge | Collect data from apps, databases, on-prem |
| Storage (Raw) | S3 (with partitioning by date/source) | Raw landing zone; immutable source of truth |
| Catalog | AWS Glue Data Catalog | Metadata store; tables schemas; partitions |
| ETL/Transform | AWS Glue (Spark jobs), Lambda | Clean, transform, enrich data |
| Storage (Curated) | S3 (Parquet/ORC format) | Transformed data ready for analysis |
| Query | Amazon Athena (SQL on S3) | Serverless; pay per query; no DB provisioning |
| Visualization | QuickSight | BI dashboards; ML-powered insights |
| Warehouse | Redshift (for complex OLAP) | For heavy analytics workloads; Redshift Spectrum queries S3 directly |

## **5\. Event-Driven Decoupled Architecture**

**Pattern:** Producers emit events → Event bus (EventBridge/SNS) → Multiple consumers process independently. Components are loosely coupled; changes to one don't affect others.

| Scenario | Pattern | Services |
| ----- | ----- | ----- |
| Order placed → notify inventory, billing, and shipping simultaneously | Fan-out | SNS → 3 SQS queues (one per service) |
| Batch processing with retry on failure | Queue \+ Worker | SQS (Standard) → ASG of workers → DLQ for failures |
| React to CloudTrail API events (e.g., non-compliant resource created) | Event rule | EventBridge rule matching event pattern → Lambda remediation |
| Process uploaded files asynchronously | S3 Event \+ Queue | S3 PUT → S3 Event Notification → SQS → Lambda/ECS worker |
| Microservice workflow with compensation | Step Functions | Step Functions state machine orchestrating Lambda steps with error handling |

## **6\. Well-Architected Framework — 6 Pillars**

| Pillar | Core Question | Key AWS Tools |
| ----- | ----- | ----- |
| Operational Excellence | How do you run and monitor systems to deliver business value? | CloudFormation (IaC), CloudWatch, AWS Config, X-Ray, Systems Manager |
| Security | How do you protect information and systems? | IAM, KMS, CloudTrail, GuardDuty, Shield, WAF, Macie, Security Hub |
| Reliability | How do you recover from failures and meet demand? | Route 53, Auto Scaling, Multi-AZ/Multi-Region, Backups, Circuit Breakers |
| Performance Efficiency | How do you use resources efficiently? | Auto Scaling, Lambda, ElastiCache, CloudFront, Graviton instances |
| Cost Optimization | How do you avoid unnecessary costs? | Reserved/Spot Instances, S3 Intelligent-Tiering, Trusted Advisor, Cost Explorer, Savings Plans |
| Sustainability | How do you minimize environmental impact? | Graviton (better performance/watt), serverless, right-sizing, Managed Services |



# **Migration Services**

| Service | Migrates | Key Facts |
| ----- | ----- | ----- |
| AWS DMS (Database Migration Service) | Databases (homogeneous or heterogeneous) | Minimal downtime; continuous replication (CDC); Schema Conversion Tool (SCT) for heterogeneous migrations (e.g., Oracle → PostgreSQL) |
| AWS SMS (Server Migration Service) | On-prem virtual machines | Lift-and-shift VMs; incremental replication; Agentless; now superseded by Application Migration Service |
| AWS Application Migration Service (MGN) | Physical, virtual, cloud servers | Continuous block-level replication; cutover with minimal downtime; replaces SMS |
| AWS Snowball Edge (Storage Optimized) | Large data volumes (TBs to PBs) offline | 80 TB usable storage; petabyte-scale data transfer; for areas with limited bandwidth |
| AWS Snowmobile | Exabyte-scale data transfer | A 45-foot shipping container truck; for \> 10 PB migrations |
| AWS DataSync | File/NFS/SMB data to AWS (online) | Scheduled sync; on-prem NFS/SMB/HDFS → S3/EFS/FSx; agent-based; faster than cp over DX |
| AWS Transfer Family | SFTP/FTP/FTPS to/from S3 or EFS | Managed SFTP server endpoint backed by S3; no protocol change needed for clients |

| 💡 TIP: Snowball decision: \< 10 PB → Snowball Edge (multiple units). \> 10 PB → Snowmobile. Rule of thumb: if it would take \> 1 week to transfer over your network, use Snowball. |
---
<br>

# **Analytics Services**

| Service | Purpose | Key Detail |
| ----- | ----- | ----- |
| Athena | Serverless SQL query over S3 | Pay per query ($5/TB scanned); use columnar formats (Parquet/ORC) and partitioning to reduce cost; no infrastructure |
| Redshift | Managed columnar data warehouse (OLAP) | MPP; columnar storage; Redshift Spectrum queries S3 in-place; RA3 nodes decouple compute from storage |
| EMR (Elastic MapReduce) | Managed Hadoop/Spark/Hive/Presto cluster | For large-scale data processing; transient clusters for batch jobs; use Spot for task nodes |
| Glue | Managed ETL + Data Catalog | Serverless Spark ETL; Crawlers auto-discover schema; Data Catalog integrates with Athena/Redshift/EMR |
| Glue Job Bookmarks | Prevent duplicate ETL data processing | Tracks state during ETL runs to record processed S3 data; prevents reprocessing old data in scheduled daily runs, drastically reducing execution time and S3 retrieval costs |
| QuickSight | Serverless BI and visualization | SPICE in-memory engine; ML-powered insights; pay per session; embeddable dashboards |
| OpenSearch Service | Search + log analytics | Elasticsearch API compatible; OpenSearch Dashboards (Kibana); anomaly detection; UEBA |
| Lake Formation | Build, secure, govern data lake on S3 | Fine-grained access control on Glue Data Catalog; column/row level security; LF-TBAC (Tag-Based Access Control) uses LF-tags on columns/tables/S3 paths for granular column-level cross-account access with low overhead |

---


# **Application Integration & Orchestration**

| Service | Purpose | Key Facts |
| ----- | ----- | ----- |
| Step Functions | Serverless workflow orchestration (state machine) | Visual workflows; Standard (1-year duration; async) vs Express (5-min; IoT/streaming); error handling, retries, parallel states |
| AppFlow | No-code SaaS data integration | Salesforce, ServiceNow, Slack → S3, Redshift; scheduled or event-driven; field-level encryption in transit |
| API Gateway | Managed API endpoint (REST, HTTP, WebSocket) | Throttling; caching; API keys; usage plans; mock integration; Cognito/Lambda authorizers |
| AWS AppSync | Managed GraphQL service | Real-time data sync; subscriptions (WebSocket); resolvers to DynamoDB, Lambda, HTTP; for mobile/web apps |
| SWF (Simple Workflow Service) | Workflow orchestration (legacy) | Prefer Step Functions; SWF used when you need human approval steps or external process integration via long-running tasks |

----

# **Other Important Services**

| Service | Purpose | SAA Exam Angle |
| ----- | ----- | ----- |
| CloudFormation | Infrastructure as Code (IaC) — JSON/YAML templates | Reproducible infrastructure; drift detection; nested stacks for complex architectures; StackSets for multi-account/region |
| CDK (Cloud Development Kit) | IaC using programming languages (TypeScript, Python, Java) | Compiles to CloudFormation; developer-friendly |
| Elastic Beanstalk | PaaS: deploy code, AWS manages infra | Developers upload code → EB handles EC2, ASG, ELB, RDS; not for complex architectures |
| OpsWorks | Managed Chef/Puppet configuration management | Legacy; used when existing Chef/Puppet tooling must be preserved |
| Systems Manager (SSM) | Operational management of EC2/on-prem instances | Run Command (execute scripts without SSH); Patch Manager; Session Manager (browser SSH no port 22); Parameter Store |
| SSM Automation (Patching with ALB) | Safely patch EC2 instances registered in an ALB target group | Use `AWS-EC2-PatchLoadBalancerInstance` runbook to deregister, patch, reboot, and re-register instances without application disruption |
| AWS Organizations | Centrally manage multiple AWS accounts | Service Control Policies (SCPs) restrict what member accounts can do; consolidated billing; OU hierarchy |
| AWS Control Tower | Set up multi-account landing zone with guardrails | Uses Organizations + SSO + Config + CloudTrail; account vending machine via Account Factory |
| AWS Trusted Advisor | Best practice checks across 5 categories | Cost optimization, performance, security, fault tolerance, service limits; Business/Enterprise support for all checks |
| AWS Cost Explorer | Visualize and analyze AWS costs | Forecasts, reserved instance recommendations, savings plans recommendations |
| AWS Budgets | Set spending alerts and take action | Alert when cost > threshold; trigger SNS or take action; budget for cost/usage/RI coverage |
| Cognito | User authentication and authorization for apps | User Pools = user directory + auth (login). Identity Pools = exchange tokens for AWS temp credentials. Often used together. |
| AWS Global Accelerator | Improve global app performance using AWS network | Static anycast IPs; routes to nearest healthy endpoint via AWS backbone; good for TCP/UDP non-HTTP traffic; vs CloudFront (HTTP/cache) |
| AWS Outposts | AWS hardware in your data center | Same AWS APIs/tools on-premises; for low-latency local data processing or data residency requirements |
| AWS Wavelength | AWS infrastructure embedded in telecom 5G networks | Ultra-low latency to mobile/connected devices; no traffic leaves telecom provider network |
| AWS Local Zones | AWS infrastructure in metro areas outside main regions | Single-digit ms latency to specific cities; extension of a parent region |

| NEXT: SECTION 11 — THE 'WHICH ONE?' QUICK REFERENCE (COMMON CONFUSIONS) |
| :---: |

---


---

# **Most-Confused Services & Concepts**

## **Global Accelerator vs CloudFront**

| Dimension | Global Accelerator | CloudFront |
| ----- | ----- | ----- |
| What it does | Routes TCP/UDP traffic globally via AWS backbone to nearest healthy endpoint | CDN that caches HTTP/HTTPS content at edge locations |
| Caching? | NO — purely routing/load balancing | YES — main feature; caches static and dynamic content |
| Protocol | Any TCP/UDP (HTTP included but not specialized) | HTTP/HTTPS/WebSocket only |
| IP Addresses | 2 static anycast IPv4 globally (fixed entry point) | Dynamic — IPs change; use domain only |
| Network Path | Traffic enters the private AWS global network at the nearest edge, significantly reducing internet jitter and network latency | Traverses internet to nearest edge location, then AWS backbone |
| Best for | Gaming (UDP), IoT, VoIP, fast regional failover, non-HTTP APIs, applications where firewall IP whitelisting is a hard constraint | Web apps, media delivery, static/dynamic web content |
| Health checks | Yes — routes away from unhealthy endpoints instantly | Via origin health checks + custom error responses |
| Exam keyword | "static IP", "non-HTTP", "gaming", "fast failover", "consistent performance", "IP whitelisting constraint" | "cache", "CDN", "reduce origin load", "static assets", "media streaming" |

## **SQS vs SNS vs Kinesis**

| Dimension | SQS | SNS | Kinesis |
| ----- | ----- | ----- | ----- |
| Pattern | Queue (pull) | Pub/Sub (push) | Stream (pull) |
| Ordering | Standard: best effort; FIFO: guaranteed | No ordering | Ordered within shard |
| Persistence | 14 days | None (push only) | 1-365 days |
| Consumers | One consumer (competing), FIFO \= one at a time | Multiple (up to 12.5M) | Multiple (unlimited with enhanced fan-out) |
| Throughput | Nearly unlimited (Standard) | Nearly unlimited | Provisioned (shards); On-Demand mode available |
| Replay | No | No | Yes |
| Use case | Worker pools, decoupling, buffering | Notifications, fan-out | Real-time analytics, click stream, IoT |


## **EBS vs EFS vs S3**

| Dimension | EBS | EFS | S3 |
| ----- | ----- | ----- | ----- |
| Type | Block storage | File storage (NFS) | Object storage |
| Attach to | Single EC2 (same AZ); io1/io2 Multi-Attach (same AZ) | Many EC2 instances across AZs (Linux) | Accessed via HTTP/HTTPS (not mountable natively) |
| Scope | Single AZ | Multi-AZ regional | Global (regional bucket, globally accessible) |
| Scaling | Manual (increase volume size) | Auto-scales | Unlimited auto-scales |
| Performance | Up to 256,000 IOPS (io2) | Throughput scales with size; Provisioned Throughput mode | High throughput; best for large objects |
| Use case | OS boot volumes, databases | Shared config files, CMS content, web serving | Backup, archives, static websites, data lake |
| Cost model | Per GB provisioned | Per GB stored (+ throughput mode option) | Per GB stored \+ requests \+ data transfer |

## **RDS Multi-AZ vs Aurora Multi-AZ vs Aurora Global DB**

| Feature | RDS Multi-AZ | Aurora Multi-AZ | Aurora Global DB |
| ----- | ----- | ----- | ----- |
| Failover time | \~1-2 min | \< 30 sec | \< 1 min (regional failover) |
| Read from standby? | No | Yes (up to 15 Aurora Replicas) | Yes (local reads in each region) |
| Replication | Sync to 1 standby in same region | 6 copies in 3 AZs (shared volume) | Async to up to 5 secondary regions |
| RPO | Near zero (sync) | Near zero (sync writes confirmed in 4/6 copies) | \< 1 second (async) |
| Scope | Single region, 2 AZs | Single region, 3 AZs | Multi-region |
| Use for | HA within region | HA \+ reads within region | Global DR \+ local reads globally |

## **CloudWatch vs CloudTrail vs Config vs Trusted Advisor**

| Service | Question it Answers | Data Source | Memory Aid |
| ----- | ----- | ----- | ----- |
| CloudWatch | How is my application PERFORMING? | Metrics, logs from AWS services and applications | Watch \= performance monitoring |
| CloudTrail | WHO called what AWS API, WHEN? | AWS API call logs (management \+ data events) | Trail \= footprints (audit trail) |
| AWS Config | Is my infrastructure COMPLIANT with rules? | Configuration item history for every AWS resource | Config \= configuration compliance |
| Trusted Advisor | Am I following AWS BEST PRACTICES? | AWS analysis of your account | Advisor \= recommendations |

## **Cognito User Pools vs Identity Pools**

| Feature | User Pool | Identity Pool |
| ----- | ----- | ----- |
| Purpose | Handles user sign-up, sign-in, MFA, password reset | Exchanges tokens for temporary AWS credentials (STS) |
| Output | JWT tokens (ID token, access token, refresh token) | AWS credentials (Access Key ID \+ Secret \+ Session Token) |
| Who are the users? | Application users (your customers) | Any authenticated user (User Pool, Google, Facebook, SAML, guest) |
| AWS service access? | No (JWT is for your app) | Yes — gives access to AWS services (S3, DynamoDB, etc.) |
| Exam scenario | "user login for web/mobile app" | "users need direct access to S3 from mobile app" |

## **Direct Connect vs VPN vs PrivateLink vs Peering**

| Option | What it is | Use when |
| ----- | ----- | ----- |
| Site-to-Site VPN | Encrypted IPSec over public internet | Fast to set up, cost-sensitive, acceptable variable latency, \< 1.25 Gbps |
| Direct Connect | Private physical connection, no internet | Consistent performance, high bandwidth (1/10/100 Gbps), compliance, low latency |
| VPC Peering | Direct VPC-to-VPC private link | Connect two VPCs (same or different account/region); non-transitive; no overlapping CIDR |
| Transit Gateway | Central hub connecting many VPCs \+ on-prem | Scale beyond peering mesh; transitive routing; simplify hub-and-spoke network |
| PrivateLink | Expose one service to consumers without full VPC access | SaaS-to-customer access; one-way service exposure; no route tables needed |
| VPC Endpoints | Private access to AWS public services from VPC | Access S3/DynamoDB (Gateway) or other AWS services (Interface) without internet |

| NEXT: SECTION 12 — PRACTICE QUESTIONS BY DOMAIN |
| :---: |

---

---

# **Exam-Style Practice Questions (But Without MCQs, So Think!)**

## **Domain 1: Design Resilient Architectures**

### **Q1**

**Question:** A company runs a critical e-commerce application on a single EC2 instance with an attached EBS volume. They want to achieve a recovery time objective (RTO) of under 5 minutes and a recovery point objective (RPO) of under 1 minute. What should a solutions architect recommend?

| 📌 NOTE: Think about what provides BOTH fast failover AND near-zero data loss before reading the answer below. |
| :---- |

**Answer:** Deploy a Multi-AZ RDS instance for the database (synchronous replication → RPO near-zero). Move the application to an EC2 Auto Scaling Group with a minimum of 2 instances across 2 AZs behind an ALB. Use Amazon Aurora instead of RDS if sub-30-second failover is needed. The ALB health checks will route around an unhealthy instance within seconds (RTO \< 5 min).

### **Q2**

**Question:** A startup stores user-uploaded images in S3. As the company grows, they need to serve images with low latency to global users, while protecting the S3 bucket from direct public access. What is the most appropriate solution?

**Answer:** Create a CloudFront distribution with an S3 origin. Enable Origin Access Control (OAC) on the CloudFront distribution and update the S3 bucket policy to allow access only from CloudFront's service principal. Disable Block Public Access settings that would conflict, but block all direct S3 access. This ensures users access content through CloudFront (low latency via edge caching) and cannot bypass it to access S3 directly.

### **Q3**

**Question:** A financial application requires database transactions that update multiple DynamoDB tables atomically — either all succeed or all fail. The application processes 2,000 transactions per second with peaks of 10,000. Which approach should the architect choose?

**Answer:** Use DynamoDB Transactions (TransactWrite) for atomic multi-table operations. Configure DynamoDB with On-Demand capacity mode to handle variable throughput without capacity planning. DynamoDB Transactions consume 2 WCUs per item write (vs 1 normally) — factor this into capacity. The On-Demand mode automatically scales for the 5x peak without throttling.

## **Domain 2: Design High-Performing Architectures**

### **Q4**

**Question:** A media streaming company needs to ingest 1 TB of clickstream data per hour from 10,000 simultaneous users and run real-time analytics to detect trending content. Which architecture handles this?

**Answer:** Use Kinesis Data Streams to ingest clickstream events in real time (provision enough shards: at 1 MB/s per shard, \~280 shards for \~280 MB/s). Use Kinesis Data Analytics (Apache Flink) to run real-time aggregations on the stream to detect trending content. Write results to DynamoDB for low-latency serving. Simultaneously use Kinesis Data Firehose to deliver raw data to S3 for long-term analytics with Athena.

### **Q5**

**Question:** A web application experiences slow database response times during peak hours due to repetitive read queries for the same product catalog data. The data changes at most once per hour. What is the most cost-effective solution?

**Answer:** Implement ElastiCache for Redis with a lazy loading (cache-aside) pattern. On cache miss, load from RDS and cache with TTL of 3600 seconds (1 hour). For the cache to be most effective, also add Read Replicas to RDS to distribute any remaining cache-miss load. This is more cost-effective than upgrading the RDS instance class.

## **Domain 3: Design Secure Architectures**

### **Q6**

**Question:** A company is deploying an application that stores sensitive customer PII in S3. They need to: (1) automatically discover what PII exists, (2) encrypt the data, (3) detect unusual access patterns, and (4) audit all access to the data. Which combination of services achieves all four?

**Answer:** (1) Amazon Macie — automatically discovers and classifies PII/sensitive data in S3. (2) S3 SSE-KMS encryption with customer-managed CMK — encrypts data and provides audit trail of decryption. (3) Amazon GuardDuty — uses ML to detect anomalous S3 access patterns. (4) AWS CloudTrail with S3 data events enabled — logs every GetObject, PutObject call with user identity, time, and source IP.

### **Q7**

**Question:** An EC2 instance in a private subnet needs to call the DynamoDB API. The company wants to ensure traffic never traverses the public internet and they do not want to pay for NAT Gateway data transfer charges. What should the architect configure?

**Answer:** Create a VPC Gateway Endpoint for DynamoDB. Gateway Endpoints are free, route traffic through the AWS network (no internet), and require updating the route table to direct DynamoDB traffic through the endpoint. This is more cost-effective than routing through a NAT Gateway, which charges for data processed.

## **Domain 4: Design Cost-Optimized Architectures**

### **Q8**

**Question:** A data analytics company runs large Apache Spark jobs nightly that process log files stored in S3. Each job runs for 3-4 hours and can be restarted if interrupted. They want the most cost-effective compute solution.

**Answer:** Use Amazon EMR with Spot Instances for task nodes (they do the heavy lifting) and On-Demand instances for the master and core nodes (they must not be interrupted). Spot Instances can save up to 90% over On-Demand. Since the jobs can tolerate interruption and restart (fault-tolerant batch processing), Spot is ideal. Use EMR Instance Fleets or Instance Groups with multiple Spot pools for better availability.

### **Q9**

**Question:** A company stores 100 TB of log files in S3 Standard. The logs are accessed frequently in the first 30 days, rarely between 30-90 days, and almost never after 90 days. After 365 days, they must retain logs for compliance but retrievals can take up to 12 hours. Design the most cost-effective S3 strategy.

**Answer:** Create an S3 Lifecycle Policy: (1) Day 0-30: S3 Standard (frequent access). (2) Day 30-90: Transition to S3 Standard-IA (infrequent access, milliseconds retrieval). (3) Day 90-365: Transition to S3 Glacier Flexible Retrieval (archives, 1min-12hrs retrieval, 90-day minimum billing period satisfied). (4) Day 365+: Transition to S3 Glacier Deep Archive (12-48hr retrieval, cheapest for compliance retention). This progressively reduces cost as access frequency drops.

### **Q10**

**Question:** A company runs a three-tier web application with ALB, EC2 application servers, and RDS MySQL. Traffic is highly predictable: 8 AM to 6 PM weekdays at full load, negligible overnight and weekends. What purchasing strategy minimizes cost while maintaining performance?

**Answer:** For EC2: Use scheduled Auto Scaling to scale in/out the ASG based on time. Purchase Reserved Instances (1-year Standard) for the minimum baseline capacity (overnight/weekend level). The peak capacity above baseline should use On-Demand (no commitment needed for predictable but variable peak). For RDS: Multi-AZ Reserved Instance (1-year) for the MySQL instance — RDS runs 24/7 regardless of traffic, so Reserved provides the best savings.

<br><br><br><br>

# **Summary: Service Selection by Problem Type**

## **I need to store...**

| If you need to store... | Use this service |
| ----- | ----- |
| Files/objects of any size, globally durable | S3 |
| Persistent block storage for one EC2 instance | EBS |
| Shared filesystem across many Linux EC2 instances | EFS |
| Shared filesystem for Windows workloads with Active Directory | FSx for Windows File Server |
| Ultra-high IOPS for HPC or ML workloads | FSx for Lustre |
| Relational data with SQL queries (managed) | RDS (MySQL/Postgres/Oracle/SQL Server/MariaDB) |
| Relational data needing high performance \+ auto-scaling | Aurora |
| Relational data with unknown/variable load (serverless SQL) | Aurora Serverless v2 |
| Non-relational key-value at any scale | DynamoDB |
| In-memory caching for microsecond reads (DynamoDB) | DAX |
| General-purpose in-memory cache with rich data types | ElastiCache for Redis |
| Simple stateless distributed cache | ElastiCache for Memcached |
| Time-series data (IoT, metrics) | Timestream |
| Graph data (social networks, fraud) | Neptune |
| Document data (MongoDB workloads) | DocumentDB |
| Data warehouse for OLAP analytics | Redshift |
| Ledger with cryptographic audit trail | QLDB |
| Search index with full-text search | OpenSearch Service |
| Secrets / credentials with auto-rotation | Secrets Manager |
| Configuration values and non-sensitive secrets | Parameter Store (SSM) |

## 

## **I need to compute...**

| If you need to compute... | Use this service |
| ----- | ----- |
| Virtual machines with full OS control | EC2 |
| Run code without managing servers (\< 15 min tasks) | Lambda |
| Run Docker containers on managed infrastructure | ECS on Fargate |
| Run Docker containers with Kubernetes (managed K8s) | EKS |
| Deploy code without managing servers/containers (PaaS) | Elastic Beanstalk |
| Batch processing at scale (HPC, ML training) | AWS Batch |
| HPC with parallel/distributed computing | EC2 (HPC cluster placement group) \+ FSx for Lustre \+ EFA |
| Run code at CDN edge locations (lightweight) | CloudFront Functions |
| Run code at CDN edge locations (heavier, full Lambda) | Lambda@Edge |
| Scientific computing, genomics, financial risk modeling | AWS Batch or EC2 with P/G instances |

## **I need to connect / network...**

| If you need to... | Use this service |
| ----- | ----- |
| Isolate workloads in a private network | VPC (with private subnets) |
| Give public internet access to EC2 in private subnet | NAT Gateway |
| Connect two VPCs directly (same or different accounts) | VPC Peering |
| Connect many VPCs in a hub-and-spoke model | Transit Gateway |
| Connect on-premises to AWS over internet (encrypted) | Site-to-Site VPN |
| Connect on-premises to AWS with dedicated line | Direct Connect |
| Expose a service to other accounts without full VPC peering | PrivateLink |
| Access S3/DynamoDB from VPC without internet (free) | VPC Gateway Endpoint |
| Access other AWS services from VPC without internet | VPC Interface Endpoint (PrivateLink) |
| Speed up global users to your application | CloudFront or Global Accelerator |
| Route DNS with health checks and traffic policies | Route 53 |
| Distribute HTTP/HTTPS traffic across EC2/containers/Lambda | ALB (Layer 7\) |
| Distribute TCP/UDP traffic with static IPs | NLB (Layer 4\) |
| Inspect traffic through virtual security appliances | GWLB (Layer 3/4) |

## **I need to secure / comply...**

| If you need to... | Use this service |
| ----- | ----- |
| Control who can do what in AWS | IAM (Users, Roles, Policies) |
| Authenticate users for your application | Cognito User Pools |
| Give app users temporary AWS credentials | Cognito Identity Pools |
| Detect threats and anomalous behavior | GuardDuty |
| Find vulnerabilities in EC2/containers/Lambda | Inspector |
| Discover PII in S3 | Macie |
| Protect web apps from attacks (XSS, SQLi) | WAF |
| Protect from DDoS attacks | Shield (Standard free; Advanced paid) |
| Centrally enforce policies across accounts | Firewall Manager |
| Aggregate security findings in one place | Security Hub |
| Audit API calls (who did what, when) | CloudTrail |
| Track resource configuration changes \+ compliance | Config |
| Encrypt data with managed keys | KMS |
| Encrypt data with hardware key isolation (FIPS 140-2 L3) | CloudHSM |
| Manage organization-wide account hierarchy | AWS Organizations \+ SCPs |
| Set up a multi-account landing zone with guardrails | Control Tower |

## 

## 

## 

## **Key Numbers to Memorize**

| Fact | Number |
| ----- | ----- |
| S3 object max size | 5 TB |
| S3 multipart required above | 5 GB (recommended above 100 MB) |
| S3 Standard Availability | 99.99% (4 nines) |
| S3 Standard Durability | 99.999999999% (11 nines) |
| Lambda max timeout | 15 minutes |
| Lambda max memory | 10 GB |
| Lambda default concurrent executions per region | 1,000 (soft limit) |
| SQS max message size | 256 KB (2 GB with Extended Client \+ S3) |
| SQS max retention | 14 days |
| SQS FIFO max throughput | 300 TPS (3,000 with batching) |
| SQS visibility timeout max | 12 hours |
| Kinesis shard write capacity | 1 MB/s or 1,000 records/sec |
| Kinesis shard read capacity | 2 MB/s per consumer (10 MB/s with enhanced fan-out) |
| DynamoDB max item size | 400 KB |
| DynamoDB max partition key per item | 2,048 bytes |
| RDS Multi-AZ failover time | \~1-2 minutes |
| Aurora failover time | \< 30 seconds |
| Aurora storage auto-grow | 10 GB increments up to 128 TB |
| Aurora max read replicas | 15 |
| VPCs per region (default) | 5 (soft limit) |
| Subnets reserved IPs by AWS | 5 (first 4 \+ last 1\) |
| VPC CIDR min/max | /28 (16 IPs) to /16 (65,536 IPs) |
| NAT Gateway max bandwidth | 100 Gbps (scales automatically) |
| EC2 Spread Placement Group max per AZ | 7 instances |
| Route 53 Multi-Value max records | 8 |
| CloudFront edge locations | 400+ |
| Max rules per Security Group | 60 inbound \+ 60 outbound (default) |

## 

## 

## 

## **Final Exam-Day Tips**

| Tip | Detail |
| ----- | ----- |
| Eliminate wrong answers first | SAA is multiple choice. Even if unsure, eliminate 2 wrong options and you have 50/50 odds. |
| 'Most cost-effective' clues | Look for: Spot (batch/fault-tolerant), Reserved (steady-state), S3 lifecycle, Lambda vs EC2, Fargate (no EC2 mgmt), Savings Plans. |
| 'Most operationally efficient' clues | Look for: managed services (RDS over EC2 DB), serverless, automation, IaC (CloudFormation), Systems Manager. |
| 'Highly available' minimum bar | \= Multi-AZ deployment. 2 AZs is HA. 3+ AZs is highly resilient. |
| Security: least privilege first | Always choose the option with minimum required permissions. Roles \> Users. No hardcoded keys ever. |
| Read the question for scope | Is it in one account? One region? Multi-region? Global users? Each scope narrows which services apply. |
| Watch out for 'existing' systems | If the question says 'existing on-prem Oracle/Rabbit/Kafka' → migration service or compatible managed service (MQ, MSK, DMS+SCT). |
| New services \= usually not the answer | SAA targets well-established services. Extremely new launches are rarely the expected answer unless clearly indicated. |

| 🧠 MNEMONIC: The 3 MOST questions in SAA: Most RESILIENT \= Multi-AZ \+ Multi-Region \+ Backups. Most PERFORMANT \= caching \+ right instance type \+ CDN \+ read replicas. Most COST-EFFECTIVE \= Spot/Reserved/Savings Plans \+ right storage tier \+ serverless where applicable. |
| :---- |

***Good luck on your AWS SAA exam\!*** **☁️**

---

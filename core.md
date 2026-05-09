# Core Technology Interview Q&A for Principal Platform Engineer

This document contains 100 interview questions and answers covering the core technology areas required for the Appian Principal Platform Engineer role.

## AWS Compute & Serverless (1-15)

**1. What are the key differences between AWS Lambda, ECS, and EKS? When would you choose one over the others?**
*Answer:* Lambda is highly event-driven and fully serverless (no infrastructure to manage). ECS is a managed container orchestration service native to AWS, simpler than Kubernetes. EKS is managed Kubernetes. Use Lambda for short-lived, event-driven tasks. Use ECS for simple containerized microservices without the overhead of managing K8s. Use EKS when you need complex orchestration, cloud-agnostic configurations, or extensive open-source K8s ecosystem integrations.

**2. How do you mitigate Lambda "cold starts"?**
*Answer:* Use Provisioned Concurrency, which keeps functions initialized and ready to respond in double-digit milliseconds. Also, optimize the deployment package (reduce dependencies), choose faster runtimes (like Go or Rust over Java/C#), and initialize heavy connections (like DB pools) outside the handler function.

**3. What is the difference between a Spot Instance, an On-Demand Instance, and a Reserved Instance?**
*Answer:* On-Demand provides compute by the second/hour with no commitment. Reserved Instances require a 1- or 3-year commitment for a significant discount. Spot Instances use spare AWS capacity at deep discounts but can be interrupted with a 2-minute warning. 

**4. How would you design an Auto Scaling Group (ASG) to be highly available and cost-effective?**
*Answer:* Span the ASG across multiple Availability Zones (AZs). Use a Mixed Instances Policy to combine On-Demand and Spot instances (e.g., base capacity On-Demand, scaling capacity Spot). Define appropriate scaling policies (Target Tracking based on CPU or custom metrics).

**5. What is AWS Fargate, and how does it compare to EC2 as a launch type for ECS/EKS?**
*Answer:* Fargate is a serverless compute engine for containers. You don't manage the underlying EC2 instances. With the EC2 launch type, you must manage, patch, and scale the EC2 worker nodes yourself. Fargate reduces operational overhead but offers less control over the host OS and can be slightly more expensive per vCPU.

**6. Explain how EC2 Placement Groups work and the different types.**
*Answer:* Placement Groups determine how instances are placed on underlying hardware. Cluster: groups instances close together for low network latency. Partition: spreads instances across logical partitions to isolate failures (good for Kafka/HDFS). Spread: strictly places instances on distinct hardware racks to minimize correlated failures.

**7. How do you gracefully shut down instances in an Auto Scaling Group?**
*Answer:* Use Lifecycle Hooks. When an instance is marked for termination, a hook puts it in a "Terminating:Wait" state. This allows you to run a script (e.g., via EventBridge and Lambda or SSM Run Command) to drain connections, save state, or deregister from custom systems before calling `CompleteLifecycleAction` to terminate it.

**8. What are AWS Step Functions, and when would you use them over chaining Lambda functions?**
*Answer:* Step Functions is a serverless orchestration service that lets you combine AWS services into visual workflows. It maintains application state. Chaining Lambdas directly leads to tight coupling and complexity in handling retries and failures. Step Functions handle complex branching, error handling, and long-running workflows seamlessly.

**9. In AWS ECS, what is the difference between a Task Definition, a Task, and a Service?**
*Answer:* A Task Definition is a blueprint describing the containers (image, CPU/RAM, ports). A Task is the instantiation of a Task Definition running on the cluster. A Service ensures a specified number of Tasks are running continuously and registers them with a load balancer.

**10. How can you securely pass secrets to an ECS or EKS container without hardcoding them?**
*Answer:* In ECS, use the `secrets` parameter in the Task Definition to pull from AWS Secrets Manager or Systems Manager Parameter Store into environment variables. In EKS, use the Secrets Store CSI Driver or tools like External Secrets Operator to mount secrets dynamically.

**11. What is an EC2 Dedicated Host vs. a Dedicated Instance?**
*Answer:* Dedicated Instances run on hardware dedicated to a single customer. Dedicated Hosts give you visibility and control over the physical server itself, which is often required for licensing compliance (e.g., BYOL for Windows Server or SQL Server).

**12. How does the AWS Nitro System improve EC2 performance?**
*Answer:* Nitro offloads virtualization functions (network, storage, security) from the host CPU to dedicated hardware components (Nitro Cards). This frees up virtually all the host's CPU and memory for the customer's instance and improves overall performance and security.

**13. What is the maximum execution time for an AWS Lambda function?**
*Answer:* 15 minutes.

**14. If a Lambda function needs to process records from a Kinesis stream, how is scaling handled?**
*Answer:* AWS automatically scales Lambda execution based on the number of shards in the Kinesis stream. By default, there is one concurrent Lambda invocation per shard.

**15. What is compute savings plan vs EC2 reserved instances?**
*Answer:* Compute Savings Plans offer more flexibility. While standard RIs are tied to a specific instance family and region, Compute Savings Plans apply automatically across instance families, sizes, AZs, regions, and even compute types (EC2, Fargate, Lambda).

## AWS Networking (16-30)

**16. What is the difference between an Application Load Balancer (ALB) and a Network Load Balancer (NLB)?**
*Answer:* ALB operates at Layer 7 (HTTP/HTTPS), supports path-based and host-based routing, and is best for web applications. NLB operates at Layer 4 (TCP/UDP), handles millions of requests per second with ultra-low latency, and provides a static IP per AZ.

**17. Explain AWS PrivateLink and its primary use case in SaaS architectures.**
*Answer:* PrivateLink allows secure connectivity between VPCs, AWS services, and on-premises networks without exposing traffic to the public internet. In SaaS, a provider can expose their service via an Endpoint Service, and a customer can connect to it via a VPC Endpoint, ensuring traffic stays on the AWS backbone.

**18. What is a VPC Transit Gateway, and how does it compare to VPC Peering?**
*Answer:* VPC Peering creates a 1:1 connection between two VPCs (not transitive). Managing many peerings becomes a complex mesh. Transit Gateway acts as a central hub-and-spoke router connecting thousands of VPCs and on-premises networks, vastly simplifying network topology.

**19. Describe the difference between a Security Group and a Network ACL.**
*Answer:* SGs operate at the instance/ENI level, are stateful (return traffic is automatically allowed), and support only allow rules. NACLs operate at the subnet level, are stateless (return traffic must be explicitly allowed), and support both allow and deny rules.

**20. How do you provide internet access to a private subnet?**
*Answer:* Deploy a NAT Gateway in a public subnet (which has a route to an Internet Gateway). Then, update the route table of the private subnet to point internet-bound traffic (0.0.0.0/0) to the NAT Gateway.

**21. What are the different routing policies available in Route 53?**
*Answer:* Simple, Weighted (distribute traffic based on weights), Latency (route to region with lowest latency), Failover (active-passive DR), Geolocation (route based on user location), Geoproximity (route based on resource location), and Multivalue Answer.

**22. How does an API Gateway differ from an ALB?**
*Answer:* API Gateway is designed for API management. It includes features like request/response transformation, throttling, caching, API keys, usage plans, and native integration with Lambda. ALB is a general-purpose L7 load balancer routing to targets like EC2 or ECS.

**23. What is VPC Flow Logs used for?**
*Answer:* VPC Flow Logs capture information about the IP traffic going to and from network interfaces in a VPC. It is used for troubleshooting connectivity issues, security analysis, and monitoring network bandwidth.

**24. Explain what an Egress-Only Internet Gateway is.**
*Answer:* It provides outbound internet access for IPv6 traffic from private subnets while preventing incoming traffic initiated from the internet. It is the IPv6 equivalent of a NAT Gateway.

**25. How do you securely connect an on-premises data center to an AWS VPC?**
*Answer:* Using AWS Site-to-Site VPN (over the internet using IPsec) or AWS Direct Connect (a dedicated, private physical network connection).

**26. What is a Gateway Endpoint vs. an Interface Endpoint (PrivateLink)?**
*Answer:* Gateway Endpoints are used only for S3 and DynamoDB; they require modifying route tables and do not use ENIs. Interface Endpoints use ENIs with private IPs in your subnets and apply to most other AWS services (EC2, KMS, SNS, etc.).

**27. How does AWS Global Accelerator differ from CloudFront?**
*Answer:* CloudFront is a CDN designed to cache static and dynamic HTTP content at edge locations. Global Accelerator improves availability and performance of applications using static IP addresses acting as a fixed entry point to applications hosted in one or more AWS Regions, utilizing the AWS global network backbone. It supports non-HTTP traffic.

**28. If an instance in a private subnet cannot reach the internet via a NAT Gateway, what should you check?**
*Answer:* 1) The private subnet's route table has 0.0.0.0/0 pointing to the NAT Gateway. 2) The NAT Gateway is in a public subnet. 3) The public subnet has a route to the Internet Gateway. 4) Security Groups and NACLs allow outbound traffic.

**29. What is SNI (Server Name Indication) on an ALB?**
*Answer:* SNI allows multiple TLS certificates to be attached to a single ALB listener, enabling the ALB to serve multiple secure websites (different domains) from the same IP address.

**30. How do you implement cross-region VPC peering?**
*Answer:* Inter-Region VPC Peering allows VPCs in different regions to communicate over the AWS global network. Setup requires accepting the peering connection in the target region and updating route tables in both VPCs.

## AWS Storage & Databases (31-40)

**31. What are the key differences between EBS, EFS, and S3?**
*Answer:* EBS is block storage attached to a single EC2 instance (like a hard drive). EFS is a managed NFS file system that can be mounted concurrently by thousands of instances. S3 is object storage accessed via HTTP APIs.

**32. Explain the different S3 storage classes.**
*Answer:* Standard (frequent access), Standard-IA (infrequent access, min 30 days), One Zone-IA (less resilient), Glacier Instant Retrieval (archive, ms access), Glacier Flexible Retrieval (minutes to hours), Glacier Deep Archive (hours, cheapest). Intelligent-Tiering automatically moves data based on access patterns.

**33. How does S3 achieve high durability?**
*Answer:* S3 redundantly stores objects on multiple devices across a minimum of three Availability Zones in an AWS Region, achieving 99.999999999% (11 9s) durability.

**34. What is the difference between RDS Multi-AZ and Read Replicas?**
*Answer:* Multi-AZ provides synchronous replication to a standby instance for high availability and disaster recovery; you cannot read from the standby. Read Replicas use asynchronous replication to scale out read traffic.

**35. How can you automate the transition of old files to cheaper storage in S3?**
*Answer:* Use S3 Lifecycle Policies to automatically transition objects to classes like Standard-IA or Glacier after a specified number of days, or expire/delete them.

**36. What is DynamoDB, and when should you use it over RDS?**
*Answer:* DynamoDB is a fully managed NoSQL key-value database providing single-digit millisecond latency at any scale. Use it for unstructured data, high-velocity read/write workloads, or serverless apps. Use RDS for complex relational joins and ACID transactions.

**37. How do you securely backup an EBS volume in another region?**
*Answer:* Take an EBS Snapshot. Then, use the AWS console, CLI, or AWS Backup to copy the snapshot to the destination region.

**38. What is the difference between EBS gp3 and io2 volume types?**
*Answer:* gp3 is general-purpose SSD where baseline performance (IOPS and throughput) is independent of volume size. io2 is Provisioned IOPS SSD designed for critical, latency-sensitive workloads needing guaranteed, high IOPS.

**39. Can you mount an EBS volume to multiple EC2 instances?**
*Answer:* Yes, using EBS Multi-Attach, but it is only supported on io1 and io2 volumes, and the instances must be in the same AZ. It also requires a cluster-aware file system (like GFS2); standard ext4/xfs won't work safely.

**40. What is Amazon Aurora Serverless?**
*Answer:* It's an on-demand, autoscaling configuration for Aurora (MySQL and PostgreSQL). It automatically starts up, shuts down, and scales capacity up or down based on application load, making it ideal for unpredictable workloads.

## Security, IAM, and Compliance (41-55)

**41. What is the difference between an IAM Role and an IAM User?**
*Answer:* An IAM User has permanent long-term credentials (password, access keys) and represents a specific person or service. An IAM Role has no permanent credentials; it is assumable by users, AWS services (like EC2), or federated identities, providing temporary security credentials.

**42. How does IAM Envelope Encryption work with KMS?**
*Answer:* AWS KMS generates a Data Key. The plaintext Data Key encrypts the actual data. Then, KMS uses a Master Key (CMK) to encrypt the Data Key. You store the encrypted data along with the encrypted Data Key. To decrypt, KMS decrypts the Data Key, which is then used to decrypt the data.

**43. What is a Service Control Policy (SCP) in AWS Organizations?**
*Answer:* SCPs are organization-wide policies that manage maximum available permissions for all accounts in an organization. They do not grant permissions; they act as a guardrail or filter (e.g., denying the ability to disable CloudTrail across all accounts).

**44. How do you implement least privilege access for an application running on EKS?**
*Answer:* Use IAM Roles for Service Accounts (IRSA). This associates an IAM Role with a Kubernetes Service Account using OIDC federation. The Pod assumes the role via the Service Account, ensuring only that specific Pod has the permissions, rather than the entire EKS node.

**45. What is the difference between AWS Secrets Manager and Systems Manager Parameter Store?**
*Answer:* Secrets Manager has native ability to automatically rotate secrets (like RDS passwords) using Lambda functions and costs money per secret. Parameter Store is a general-purpose key-value store, mostly free (for standard tiers), but does not natively auto-rotate secrets.

**46. How do you secure data at rest and data in transit?**
*Answer:* At rest: Use AWS KMS to encrypt EBS volumes, S3 buckets, RDS databases, etc. In transit: Enforce TLS/SSL using certificates from AWS Certificate Manager (ACM) on ALBs, API Gateways, and CloudFront.

**47. What is AWS CloudTrail vs AWS Config?**
*Answer:* CloudTrail records API activity and user actions (Who did what and when?). AWS Config records configuration changes to AWS resources over time and evaluates them against compliance rules (What changed and is it compliant?).

**48. How do you grant a third-party vendor access to your AWS account securely?**
*Answer:* Create an IAM Role for cross-account access. Provide the vendor with the Role ARN. Enforce the use of an `ExternalId` in the trust policy to prevent the "confused deputy" problem. Do not use IAM Users/Access Keys.

**49. Explain the concept of the "Confused Deputy" problem.**
*Answer:* It occurs when an entity with permission to perform an action is coerced by a less-privileged entity to perform that action. In AWS cross-account roles, using an `ExternalId` ensures the role is only assumed when the specific external ID associated with the 3rd party is provided.

**50. What is AWS WAF and what does it protect against?**
*Answer:* Web Application Firewall protects web applications from common web exploits (like SQL injection, cross-site scripting (XSS)) and bots. It integrates with CloudFront, ALB, and API Gateway.

**51. How would you design an environment to meet SOC2 or FedRAMP compliance?**
*Answer:* Implement strict boundary protection (VPCs, PrivateLink), comprehensive audit logging (CloudTrail, VPC Flow Logs sent to a secure, immutable S3 bucket), continuous configuration monitoring (AWS Config), at-rest and in-transit encryption (KMS/FIPS 140-2 endpoints), and centralized identity management.

**52. What is AWS GuardDuty?**
*Answer:* A continuous threat detection service that analyzes CloudTrail, VPC Flow Logs, and DNS logs to identify unauthorized or malicious activity (e.g., crypto-mining, compromised instances).

**53. Describe the principle of "Defense in Depth" in AWS.**
*Answer:* Applying security controls at multiple layers. For a web app: WAF at the edge (CloudFront), Security Groups on the ALB, private subnets for EC2, NACLs, IAM instance profiles, and database encryption. If one layer fails, others provide protection.

**54. What is a Permission Boundary in IAM?**
*Answer:* An advanced IAM feature that uses a managed policy to set the maximum permissions that an identity-based policy can grant to an IAM entity. It prevents privilege escalation.

**55. How do you securely connect developer laptops to an AWS private subnet without public IPs?**
*Answer:* Use AWS Client VPN, or set up AWS Systems Manager (SSM) Session Manager to access instances via the AWS backbone without opening inbound SSH ports.

## Infrastructure as Code & Terraform (56-70)

**56. What is Terraform State, and why is remote state important?**
*Answer:* State maps real-world resources to your configuration. Remote state (e.g., stored in S3) allows teams to collaborate safely, ensures state is encrypted, and provides a single source of truth.

**57. How do you prevent concurrent executions from corrupting Terraform state?**
*Answer:* Use state locking. If using an S3 backend, configure a DynamoDB table. Terraform will acquire a lock in DynamoDB before running and release it after, preventing parallel `terraform apply` runs.

**58. How do you handle Terraform drift (when manual changes are made in the console)?**
*Answer:* Run `terraform plan` to detect drift. To fix it, either revert the manual change in the console, or update the Terraform code to match the new reality and run `terraform apply`.

**59. What are Terraform Modules?**
*Answer:* Modules are self-contained packages of Terraform configurations that manage a collection of related resources. They promote reusability, organization, and standardization across environments.

**60. What is the `count` vs `for_each` meta-argument in Terraform?**
*Answer:* Both create multiple instances of a resource. `count` uses an index (0, 1, 2) and if you remove an item from the middle of a list, Terraform may destroy and recreate subsequent items. `for_each` uses a map or set of strings, creating resources keyed by those values, making it safer for additions/removals.

**61. How do you import existing infrastructure into Terraform?**
*Answer:* Use the `terraform import` command (e.g., `terraform import aws_instance.web i-12345678`). You must also write the corresponding resource block in your `.tf` files.

**62. What is the purpose of `terraform taint` (or `-replace` in modern Terraform)?**
*Answer:* It marks a resource for forced destruction and recreation on the next `terraform apply`, useful if a resource is degraded or failed provisioners.

**63. What are Terraform Provisioners, and when should you use them?**
*Answer:* Provisioners execute scripts (local-exec, remote-exec) on a local machine or remote server during resource creation. HashiCorp recommends using them as a last resort; prefer configuration management tools (Ansible) or cloud-init/user-data.

**64. Explain Terraform Workspaces.**
*Answer:* Workspaces allow you to manage multiple states associated with a single configuration directory (e.g., dev, staging, prod). However, for completely isolated environments, using separate directories with different backend configurations is often preferred over workspaces.

**65. How do you manage secrets in Terraform?**
*Answer:* Never hardcode them. Use environment variables (`TF_VAR_db_password`), external secret managers (AWS Secrets Manager data sources), or tools like HashiCorp Vault. Ensure state files are encrypted as they store secrets in plain text.

**66. What is the `null_resource` used for?**
*Answer:* It acts like a standard resource but takes no action. It is often used with provisioners or local-exec to run scripts when a certain trigger changes.

**67. How do you test Terraform code?**
*Answer:* Use static analysis tools like `tfsec` or `checkov` for security scanning. Use `tflint` for linting. For integration testing, use tools like `Terratest` (Go) to deploy the code, assert conditions, and tear it down.

**68. Explain `data` sources in Terraform.**
*Answer:* Data sources allow Terraform to fetch information defined outside of Terraform or computed by another Terraform configuration (e.g., looking up an existing AMI ID or a VPC ID).

**69. What is GitOps, and how does it relate to IaC?**
*Answer:* GitOps uses Git repositories as the single source of truth to deliver infrastructure and applications. IaC files are stored in Git; CI/CD tools or automated agents (like ArgoCD) continuously sync the real-world infrastructure to match the Git state.

**70. What are AWS CloudFormation StackSets?**
*Answer:* StackSets let you deploy CloudFormation stacks across multiple AWS accounts and regions with a single operation, useful for organization-wide baseline deployments.

## Kubernetes & EKS (71-85)

**71. Describe the lifecycle of a Pod.**
*Answer:* Pending (accepted, downloading images), Running (at least one container is running), Succeeded (all containers exited with 0), Failed (a container exited with non-zero), or Unknown (apiserver can't communicate with the node).

**72. What is the difference between a Deployment, StatefulSet, and DaemonSet?**
*Answer:* Deployments manage stateless applications (replicas are interchangeable). StatefulSets manage stateful applications (pods get sticky identities, ordered deployment/scaling). DaemonSets ensure a copy of a pod runs on every eligible node (good for logging/monitoring agents).

**73. How does a Kubernetes Service work, and what are the types?**
*Answer:* A Service provides a stable IP/DNS for a set of Pods. ClusterIP (internal only), NodePort (exposes port on each Node's IP), LoadBalancer (provisions cloud LB like AWS ALB/NLB), ExternalName (maps to a CNAME).

**74. What is the AWS VPC CNI for EKS?**
*Answer:* It assigns secondary IP addresses from the VPC directly to Kubernetes Pods. This means Pods get native AWS IP addresses and can be governed by VPC routing and Security Groups directly.

**75. How do you expose an HTTP application to the internet in EKS?**
*Answer:* Use an Ingress Controller (like AWS Load Balancer Controller). You define an Ingress resource with routing rules, and the controller automatically provisions an AWS Application Load Balancer (ALB) to route traffic to the Pods.

**76. What is the difference between Cluster Autoscaler and Karpenter?**
*Answer:* Cluster Autoscaler works with AWS Auto Scaling Groups to add/remove nodes based on pending pods. Karpenter is a faster, next-gen autoscaler that bypasses ASGs, provisioning compute directly to match pod requirements (right-sizing nodes dynamically).

**77. Explain Kubernetes RBAC.**
*Answer:* Role-Based Access Control determines who can do what in the cluster. You define a Role (namespace-level) or ClusterRole (cluster-level) with permissions, and bind it to a User or ServiceAccount using a RoleBinding or ClusterRoleBinding.

**78. What are Custom Resource Definitions (CRDs)?**
*Answer:* CRDs extend the Kubernetes API, allowing you to define custom objects. Operators use CRDs to manage complex third-party software (like a database operator) using native K8s tooling.

**79. How do Liveness, Readiness, and Startup probes differ?**
*Answer:* Liveness checks if the container is running; if it fails, Kubelet restarts it. Readiness checks if the container is ready to accept traffic; if it fails, it's removed from the Service endpoints. Startup checks if an application has initialized (delaying other probes until it passes).

**80. How do you perform a zero-downtime deployment in Kubernetes?**
*Answer:* Use a Deployment with a `RollingUpdate` strategy. Set `maxUnavailable` to 0 or a low percentage, and define proper Readiness probes so K8s only terminates old pods when new pods are successfully handling traffic.

**81. What is an Init Container?**
*Answer:* A container that runs and completes before the main application containers start. Useful for running setup scripts, migrations, or waiting for a database to be ready.

**82. How do you handle persistent storage in K8s?**
*Answer:* Using Persistent Volumes (PV) and Persistent Volume Claims (PVC). In AWS, the EBS CSI driver dynamically provisions an EBS volume when a PVC is created.

**83. Explain Helm and why it's used.**
*Answer:* Helm is the package manager for K8s. It packages YAML manifests into "Charts", supports templating, and manages application releases/rollbacks, making complex application deployment repeatable.

**84. What are Taints and Tolerations?**
*Answer:* Taints are applied to Nodes to repel Pods. Tolerations are applied to Pods to allow them to schedule on tainted nodes. Used for dedicating nodes to specific workloads (e.g., GPU nodes).

**85. What are Node Affinity and Pod Affinity?**
*Answer:* Node Affinity constrains which nodes a pod can be scheduled on (e.g., only nodes in us-east-1a). Pod Affinity/Anti-Affinity constrains scheduling based on the labels of *other pods* already running on the node (e.g., keeping frontend and backend pods close, or spreading web pods apart).

## CI/CD, GitOps & Automation (86-95)

**86. Explain Blue/Green vs. Canary deployments.**
*Answer:* Blue/Green runs two identical environments; traffic is switched entirely from old (blue) to new (green) at once. Canary rolls out the new version to a small subset of users (e.g., 5%) to monitor errors before fully scaling out.

**87. How does ArgoCD work in a GitOps workflow?**
*Answer:* ArgoCD is a Kubernetes controller that continuously monitors a Git repository. When changes are pushed to Git (the desired state), ArgoCD automatically syncs and applies them to the cluster (the live state).

**88. What is immutable infrastructure?**
*Answer:* Servers or containers are never modified after deployment. If an update is needed, a new image is built, deployed, and replaces the old one. This prevents configuration drift and ensures consistency.

**89. How do you secure a CI/CD pipeline?**
*Answer:* Implement branch protections, require code reviews, use short-lived credentials (OIDC instead of permanent AWS keys), integrate SAST/DAST tools, scan container images for vulnerabilities, and sign artifacts.

**90. What is an AWS CodePipeline?**
*Answer:* A fully managed continuous delivery service that helps automate release pipelines for fast and reliable application and infrastructure updates.

**91. How would you handle a database schema migration in an automated CI/CD pipeline?**
*Answer:* Use migration tools (Flyway, Liquibase). Run the migration as a pre-deployment step or an Init Container. Ensure migrations are backward compatible so that both old and new code can run concurrently during the rollout.

**92. What are the benefits of using GitHub Actions for CI/CD?**
*Answer:* It integrates natively with GitHub repositories, provides reusable workflows, hosts runners, and has a massive marketplace of pre-built actions.

**93. What is Ansible and how is it different from Terraform?**
*Answer:* Ansible is primarily a Configuration Management tool used to configure the OS and software *inside* existing machines. Terraform is a provisioning tool used to create the infrastructure itself. They are often used together.

**94. What is a "shift-left" approach in DevSecOps?**
*Answer:* Integrating security checks (linting, vulnerability scanning, secret detection) early in the software development lifecycle (e.g., on the developer's laptop or in the PR phase) rather than waiting until production deployment.

**95. How do you measure CI/CD performance (DORA metrics)?**
*Answer:* Deployment Frequency, Lead Time for Changes, Mean Time to Recovery (MTTR), and Change Failure Rate.

## Architecture & Platform Engineering (96-100)

**96. What is the difference between RTO and RPO in Disaster Recovery?**
*Answer:* RTO (Recovery Time Objective) is the maximum acceptable downtime (how long it takes to restore). RPO (Recovery Point Objective) is the maximum acceptable data loss (measured in time, e.g., backups every 1 hour means RPO is 1 hour).

**97. How do you design a multi-tenant SaaS application?**
*Answer:* There are three main models: Silo (separate infrastructure per tenant - highest isolation, highest cost), Pool (shared infrastructure and DB, tenant ID separates data - highest scale, lower isolation), and Bridge (hybrid, e.g., shared compute, separate DB schemas). The choice depends on regulatory requirements and cost.

**98. What is the role of an Internal Developer Platform (IDP)?**
*Answer:* An IDP abstracts complex infrastructure via self-service portals or APIs. It enables developers to provision environments, deploy code, and monitor apps without needing to understand Terraform, Kubernetes, or AWS deeply, thereby increasing developer velocity.

**99. How do you approach observability in a distributed microservices architecture?**
*Answer:* Implement the three pillars: Metrics (Prometheus/CloudWatch), Logs (ELK/FluentBit/CloudWatch Logs), and Traces (AWS X-Ray/Jaeger). Ensure correlation IDs are passed across microservices to trace requests end-to-end.

**100. Tell me about a time you had to optimize AWS costs on a large scale.**
*Answer:* (This is a behavioral prep question). Focus on right-sizing instances, deleting unattached EBS volumes/EIPs, transitioning S3 data to colder tiers, implementing Compute Savings Plans/Spot instances, and setting up AWS Budgets and Cost Anomaly Detection.

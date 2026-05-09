# 200 Amazon EKS & Kubernetes Interview Questions

## EKS Fundamentals & Control Plane (1-40)
**1. What is EKS?** Amazon Elastic Kubernetes Service, a managed Kubernetes service.
**2. Who manages the EKS control plane?** AWS fully manages the control plane (API server, etcd) across multiple AZs.
**3. How is the control plane priced?** $0.10 per hour per cluster.
**4. What is EKS Anywhere?** Allows running EKS on on-premises infrastructure (VMware/bare metal).
**5. What is EKS Distro?** The same open-source Kubernetes distribution deployed by EKS, available for download.
**6. Can you access the EKS etcd directly?** No, it's fully managed and hidden by AWS.
**7. How are control plane logs accessed?** By enabling control plane logging to CloudWatch Logs.
**8. What log types can you enable?** API, audit, authenticator, controllerManager, scheduler.
**9. How do you upgrade an EKS cluster?** Update the control plane via AWS, update add-ons, then update node groups.
**10. Can you downgrade an EKS cluster?** No, Kubernetes downgrades are not supported.
**11. What is the SLA for EKS?** 99.95% for the EKS API server endpoint.
**12. What happens if the API server goes down?** Existing pods keep running, but cluster management (kubectl) fails.
**13. What is an EKS Add-on?** Operational software (like CoreDNS, VPC CNI) managed by the EKS API.
**14. What are the default EKS add-ons?** VPC CNI, CoreDNS, and kube-proxy.
**15. Difference between public and private EKS endpoints?** Public allows internet access to the API server; private restricts access to the VPC.
**16. Can a cluster have both public and private endpoints?** Yes, this is often the recommended balance of security and usability.
**17. What CIDR block does the control plane use?** AWS provisions a dedicated VPC for the control plane, peering it with your VPC via ENIs.
**18. What is the maximum number of clusters per region?** Default limit is 100 per region per account.
**19. How do you authenticate to EKS?** Using AWS IAM credentials via `aws-iam-authenticator` or the AWS CLI.
**20. Where is the mapping between IAM and Kubernetes RBAC stored?** In the `aws-auth` ConfigMap in the `kube-system` namespace.
**21. Who has access to a newly created cluster?** Only the IAM entity (User/Role) that created the cluster.
**22. How do you add an admin to a cluster?** Add their IAM ARN to the `aws-auth` ConfigMap mapping them to `system:masters`.
**23. What is EKS Local Zones support?** Running EKS worker nodes in AWS Local Zones for ultra-low latency.
**24. What is Outposts support for EKS?** Running the entire EKS cluster (control and data plane) on an AWS Outpost in your data center.
**25. How do you backup an EKS cluster?** Use Velero to backup K8s manifests to S3 and snapshot persistent volumes.
**26. Can you change the VPC of an existing EKS cluster?** No, you must create a new cluster and migrate workloads.
**27. What version of K8s does EKS run?** Upstream Kubernetes, usually supporting the 3 or 4 most recent versions.
**28. How long does an EKS K8s version stay supported?** Typically 14 months after standard release, plus Extended Support for a fee.
**29. What is EKS Extended Support?** Allows running older K8s versions for an additional 12 months (total 26 months) for an extra cost.
**30. How do you deploy resources to a new EKS cluster?** Using `kubectl`, Helm, or GitOps tools like ArgoCD/Flux.
**31. What is AWS Systems Manager (SSM) relative to EKS?** Used to securely connect to worker nodes without SSH keys.
**32. What is AWS RAM in relation to EKS?** AWS Resource Access Manager can share subnets across accounts, allowing EKS nodes in different accounts to share a VPC.
**33. How do you enforce tags on EKS clusters?** Use AWS Tag Policies or Service Control Policies (SCPs).
**34. Can EKS use custom AMIs for the control plane?** No, the control plane is a managed black box.
**35. What is the cluster security group?** A security group automatically created by EKS to allow communication between the control plane and worker nodes.
**36. How do you encrypt the EKS control plane?** EKS control plane volumes are encrypted by default by AWS.
**37. How to enable envelope encryption for Secrets?** Specify an AWS KMS key during cluster creation to encrypt K8s secrets in etcd.
**38. Can you enable KMS encryption on an existing EKS cluster?** No, it must be enabled at cluster creation.
**39. What is the default node OS for EKS?** Amazon Linux 2 (AL2) or AL2023.
**40. Are multiple AZs required for EKS?** Yes, EKS requires subnets in at least two AZs during creation.

## Compute & Node Management (41-80)
**41. What are EKS Managed Node Groups?** AWS-managed EC2 Auto Scaling Groups that automate node provisioning and lifecycle.
**42. What is a Self-Managed Node Group?** Standard EC2 ASGs where you manually manage the AMI, bootstrapping, and updates.
**43. What is AWS Fargate for EKS?** Serverless compute for pods; no underlying EC2 instances to manage.
**44. How does Fargate pricing work?** Billed per second based on the vCPU and memory requested by the pod.
**45. Does Fargate support DaemonSets?** No, because there are no nodes to run the daemons on.
**46. Does Fargate support GPUs?** No, GPUs are not supported on Fargate.
**47. What is a Fargate Profile?** Specifies which pods (based on namespace/labels) should be scheduled on Fargate.
**48. What is Karpenter?** A high-performance, open-source Kubernetes cluster autoscaler built by AWS.
**49. How is Karpenter different from Cluster Autoscaler (CA)?** CA modifies ASGs; Karpenter provisions EC2 instances directly based on pending pod requirements.
**50. Why is Karpenter faster than CA?** It bypasses ASGs and provisions right-sized instances directly via the EC2 Fleet API.
**51. What is a Karpenter Provisioner/NodePool?** Defines constraints (instance types, zones, capacity types) for the nodes Karpenter can launch.
**52. Can you run Spot instances on EKS?** Yes, via Managed Node Groups, Self-Managed ASGs, or Karpenter.
**53. How do you handle Spot interruptions gracefully?** Install the AWS Node Termination Handler to cordon/drain nodes upon receiving the 2-minute warning.
**54. Does Karpenter handle Spot interruptions?** Yes, Karpenter natively listens to EC2 Spot interruption events and drains nodes.
**55. What is Bottlerocket?** A minimal, Linux-based open-source OS by AWS optimized for running containers.
**56. Why use Bottlerocket over Amazon Linux 2?** Smaller attack surface, faster boot times, and transactional updates.
**57. Can you SSH into a Bottlerocket node?** Not directly; you must use an admin container accessed via SSM Session Manager.
**58. How do you pass user-data to an EKS Managed Node Group?** Using an EC2 Launch Template attached to the node group.
**59. What is the `/etc/eks/bootstrap.sh` script?** The script run via user-data to join a self-managed node to the EKS cluster.
**60. How do you scale EKS nodes to zero?** Set the ASG minimum to 0 (for CA), or use Karpenter which naturally scales empty nodes to zero.
**61. What is Node consolidation in Karpenter?** Karpenter continuously monitors nodes and replaces underutilized nodes with cheaper/smaller ones to save costs.
**62. How are worker nodes authenticated to the cluster?** Using the Node IAM Role, which is mapped to the `system:node` K8s group.
**63. What is the minimum IAM policy required for an EKS node?** `AmazonEKSWorkerNodePolicy`, `AmazonEC2ContainerRegistryReadOnly`, `AmazonEKS_CNI_Policy`.
**64. Can Fargate pods use EBS volumes?** No, Fargate only supports EFS for persistent storage.
**65. How do you assign Elastic IPs to EKS nodes?** Not recommended, but possible via Launch Templates or custom networking scripts. Use NAT Gateways for outbound IPs.
**66. What is a Launch Template?** An AWS resource containing configuration information to launch EC2 instances (AMI, instance type, user-data).
**67. How do you perform a rolling update on Managed Node Groups?** Update the Node Group version in AWS; EKS automatically creates new nodes, drains old ones, and terminates them.
**68. What happens if a pod cannot be drained during a node update?** EKS will wait up to 15 minutes, then forcefully terminate the node (unless PodDisruptionBudgets block it entirely).
**69. How do you use ARM (Graviton) processors in EKS?** Specify Graviton instance types (e.g., `m6g`) in your Node Group and ensure your container images are multi-arch.
**70. What is multi-arch image support?** A single Docker manifest that contains both x86 and ARM image variants.
**71. Can you mix x86 and ARM nodes in the same cluster?** Yes, use nodeSelectors or taints to ensure pods land on the correct architecture.
**72. What are AWS Outposts instances?** Specific EC2 instance types available on AWS Outposts for local deployment.
**73. What is the EKS optimized AMI?** An AMI maintained by AWS containing Docker/containerd, kubelet, and necessary AWS binaries.
**74. What is Containerd?** The industry-standard container runtime used by EKS (Docker is deprecated).
**75. How do you debug a failed Managed Node Group creation?** Check the EC2 Auto Scaling Group activities and CloudTrail logs for permissions issues.
**76. Can you attach a GPU to a Fargate pod?** No.
**77. How do you expose a GPU to a pod on an EC2 node?** Use a GPU instance type, the EKS GPU AMI, and deploy the NVIDIA device plugin DaemonSet.
**78. What is the AWS Inferentia chip?** AWS custom silicon for machine learning inference; supported in EKS via `Inf1`/`Inf2` instances and the Neuron device plugin.
**79. How do you restrict which AZs a Node Group uses?** By specifying only specific subnets when creating the Node Group.
**80. What is a pod topology spread constraint?** Ensures pods are evenly distributed across AZs or nodes to maximize availability.

## Networking (VPC CNI, LBs, App Mesh) (81-120)
**81. What is the Amazon VPC CNI?** The default networking plugin for EKS; it assigns VPC IP addresses directly to pods.
**82. How does VPC CNI assign IPs?** It attaches Elastic Network Interfaces (ENIs) to EC2 nodes and assigns secondary IPs from the ENI to pods.
**83. What is the biggest drawback of VPC CNI?** IP address exhaustion; dense clusters can rapidly consume all IPs in a subnet.
**84. How do you solve VPC IP exhaustion in EKS?** Enable "Custom Networking" (using secondary CIDRs) or enable "Prefix Delegation".
**85. What is Prefix Delegation?** VPC CNI assigns a full /28 IP prefix (16 IPs) to an ENI instead of a single IP, vastly increasing node pod density.
**86. Does Fargate use the VPC CNI?** Yes, each Fargate pod gets a dedicated ENI and VPC IP address.
**87. Can you use Calico with EKS?** Yes, either as the primary CNI or alongside VPC CNI just for Network Policies.
**88. Can you use Cilium with EKS?** Yes, it can completely replace `kube-proxy` and VPC CNI, offering eBPF-based networking.
**89. What is the AWS Load Balancer Controller (LBC)?** A controller that provisions ALBs for K8s `Ingress` and NLBs for `Service` of type `LoadBalancer`.
**90. Difference between `instance` and `ip` target types in LBC?** `instance` routes to EC2 NodePorts; `ip` routes directly to the Pod IP (more efficient).
**91. How does the ALB Controller know which subnets to use?** It looks for subnets tagged with `kubernetes.io/role/elb` (public) or `kubernetes.io/role/internal-elb` (private).
**92. How do you attach an ACM SSL Certificate to an EKS Ingress?** By adding the annotation `alb.ingress.kubernetes.io/certificate-arn` to the Ingress resource.
**93. What is Security Groups for Pods?** Allows assigning a specific AWS Security Group directly to a pod (via an ENI), rather than relying on the node's Security Group.
**94. What is the limit of Security Groups for Pods?** It reduces the maximum number of pods per node because each pod requires a dedicated ENI.
**95. How does EKS handle outbound internet traffic for private pods?** Traffic routes via the VPC CNI to the node, then follows the VPC route table to a NAT Gateway.
**96. What happens if a pod needs to communicate with S3?** Traffic normally goes via NAT Gateway. To save costs, set up a VPC Gateway Endpoint for S3.
**97. What is AWS App Mesh?** AWS's managed service mesh, built on Envoy, that integrates natively with EKS.
**98. Why use a Service Mesh in EKS?** For mTLS between pods, advanced traffic routing (canary deployments), and rich observability (tracing).
**99. How do you do cross-cluster communication?** Peer the VPCs and use internal Load Balancers, or use a multi-cluster Service Mesh like Istio.
**100. What is the Cloud Map integration in EKS?** Uses AWS Cloud Map (service discovery) instead of CoreDNS, useful for hybrid EKS/ECS architectures.
**101. Why would CoreDNS loop or fail?** Usually due to VPC DNS resolution issues, or the CoreDNS pods lacking network access to the API server.
**102. Can you assign an Elastic IP to an ALB created by Ingress?** No, ALBs use dynamic IPs. To use an EIP, use a Network Load Balancer (NLB).
**103. How to preserve the client source IP in EKS?** Set `externalTrafficPolicy: Local` on the Service, or use target-type `ip` with NLB/ALB.
**104. What is `externalTrafficPolicy: Local`?** Bypasses kube-proxy load balancing; traffic only routes to pods on the node receiving the traffic, preserving source IP but potentially causing imbalanced load.
**105. How do you restrict traffic between pods in EKS?** Use Kubernetes Network Policies (requires the VPC CNI Network Policy agent or Calico).
**106. What is WAF integration with EKS?** You can attach AWS WAF to an ALB provisioned by the Load Balancer Controller using annotations.
**107. Can EKS pods use IPv6?** Yes, EKS supports IPv6 clusters. Pods get IPv6 addresses, alleviating IPv4 exhaustion.
**108. In an IPv6 cluster, how do pods reach the IPv4 internet?** Using NAT64 and DNS64, handled automatically by AWS.
**109. What is SNAT (Source Network Address Translation) in VPC CNI?** External traffic leaves the node using the node's primary IP, not the pod's IP.
**110. When would you disable SNAT in VPC CNI?** When peering with on-premises networks so that on-prem firewalls see the actual pod IPs.
**111. How do you share an ALB across multiple Ingress resources?** Use the `alb.ingress.kubernetes.io/group.name` annotation.
**112. What is TargetGroupBinding?** A CRD from the ALB Controller that allows you to attach pods directly to an existing, pre-provisioned AWS Target Group.
**113. What port does kube-proxy use for health checks?** 10256.
**114. How do you restrict ALB access to specific IPs?** Use the `alb.ingress.kubernetes.io/inbound-cidrs` annotation or WAF.
**115. Can an NLB terminate TLS?** Yes, NLB supports TLS listeners using ACM certificates.
**116. How do you setup gRPC routing in EKS?** Use an ALB with the `alb.ingress.kubernetes.io/backend-protocol-version: GRPC` annotation.
**117. What is Multus CNI?** A meta-plugin that allows attaching multiple network interfaces to a single pod.
**118. Use case for Multus in EKS?** Telecom workloads (5G cores) requiring dedicated, high-throughput SR-IOV interfaces separate from the management network.
**119. How do you view VPC CNI logs?** By checking the `aws-node` daemonset logs in the `kube-system` namespace.
**120. Does EKS support AWS Global Accelerator?** Yes, you can place a Global Accelerator in front of an ALB/NLB provisioned by EKS.

## Identity, Security & Compliance (121-160)
**121. What is IRSA?** IAM Roles for Service Accounts. Allows pods to assume AWS IAM roles.
**122. How does IRSA work?** EKS hosts an OIDC provider. Pods get a JWT token; the AWS SDK exchanges the JWT for temporary STS credentials.
**123. What is EKS Pod Identity?** A newer, simpler alternative to IRSA that uses an EKS-managed agent instead of OIDC to deliver credentials to pods.
**124. Why prefer Pod Identity over IRSA?** It's easier to set up, requires no OIDC configuration, and simplifies cross-account role assumption.
**125. What happens if a pod doesn't use IRSA/Pod Identity?** It defaults to using the broad IAM role attached to the EC2 worker node (a massive security risk).
**126. How do you block pods from using the EC2 metadata service (IMDS)?** Require IMDSv2 with a hop limit of 1 on the EC2 instances, preventing pods from reaching `169.254.169.254`.
**127. What is AWS KMS Envelope Encryption in EKS?** Encrypting the Kubernetes etcd database (which holds Secrets) using an AWS KMS key.
**128. How do you secure cluster endpoints?** Set the endpoint to Private-only, or use Public with strict CIDR whitelisting.
**129. What is Amazon GuardDuty for EKS?** A threat detection service that analyzes EKS audit logs to find malicious activity (e.g., unauthorized access to secrets).
**130. What is GuardDuty EKS Runtime Monitoring?** Uses an eBPF agent deployed to nodes to detect runtime threats inside containers (like reverse shells or crypto-mining).
**131. How do you manage K8s secrets using AWS services?** Use the Secrets Store CSI Driver with the AWS Provider to mount Secrets Manager/Parameter Store secrets as volumes.
**132. What is the `aws-auth` ConfigMap problem?** It is highly sensitive and prone to manual syntax errors that can lock administrators out of the cluster.
**133. What is the replacement for `aws-auth` ConfigMap?** EKS Access Entries (Access Management API), which allows granting access via AWS API/Terraform without editing the ConfigMap.
**134. What is Kyverno?** A policy engine designed specifically for Kubernetes. It can validate, mutate, and generate configurations.
**135. What is OPA Gatekeeper?** A policy engine using the Rego language to enforce custom admission control policies in EKS.
**136. How do you ensure only approved container images run in EKS?** Use OPA/Kyverno to block images not originating from your specific ECR registry account ID.
**137. How to scan images in ECR?** Enable ECR Basic (Clair) or Enhanced (Inspector) scanning to detect CVEs automatically on push.
**138. How to enforce Pod Security Standards (PSS)?** Use the built-in Pod Security Admission controller by labeling namespaces (e.g., `pod-security.kubernetes.io/enforce: restricted`).
**139. What is AWS Network Firewall?** Can be deployed in a transit VPC to inspect and filter all outbound traffic from EKS nodes.
**140. How to securely connect a developer laptop to private EKS?** Use AWS Client VPN, Direct Connect, or an SSM Bastion host.
**141. What is an OIDC Identity Provider?** OpenID Connect. EKS uses it to integrate with external identity systems like Azure AD or Okta for kubectl access.
**142. How to implement zero-trust in EKS?** Use a Service Mesh (mTLS between all pods), Network Policies (default deny), IRSA (least privilege), and read-only root filesystems.
**143. What is a mutating admission webhook?** Intercepts requests to the API server and modifies them before saving (e.g., automatically injecting the Envoy sidecar).
**144. What is a validating admission webhook?** Intercepts requests and rejects them if they don't meet policies (e.g., rejecting pods that request excessive CPU).
**145. How to rotate KMS keys used by EKS?** Enable automatic key rotation in AWS KMS; EKS will automatically use the new key material.
**146. How do you audit `kubectl exec` sessions?** EKS Audit Logs capture the API request, but actual keystrokes require third-party tools like Teleport.
**147. What is CIS Kubernetes Benchmark?** A set of security guidelines. You can use tools like `kube-bench` to check EKS node compliance (though AWS manages the control plane).
**148. How do you restrict cross-namespace communication?** Apply a default-deny Network Policy in every namespace, then explicitly allow required traffic.
**149. Can you use AWS Cognito with EKS?** Yes, via an ALB. The ALB can authenticate users via Cognito before routing traffic to the EKS pods.
**150. What is `automountServiceAccountToken: false`?** Prevents Kubernetes from automatically injecting the API token into a pod that doesn't need to talk to the K8s API.
**151. Why drop Linux capabilities?** To remove unnecessary root privileges (like `CAP_NET_ADMIN` or `CAP_SYS_ADMIN`) to minimize blast radius.
**152. What is AWS IAM Access Analyzer?** Can be used to identify EKS IAM roles (IRSA) that are shared with external principals or have overly permissive policies.
**153. How to secure the kubelet API?** In EKS, the kubelet API is secured by default using webhook authentication and authorization.
**154. What is SOX/SOC2 compliance on EKS?** Requires strict audit logging (CloudTrail + EKS Audit Logs), encryption at rest/transit, and robust RBAC/IAM segregation.
**155. Can EKS pods share memory?** Yes, using IPC, but it's restricted by default. Use SecurityContext to enable if required (rare).
**156. What is Seccomp?** Secure Computing Mode. Profiles that restrict the system calls a container can make to the Linux kernel.
**157. What is AppArmor?** A Linux kernel security module that restricts programs' capabilities via profiles (similar to Seccomp).
**158. How do you use private ECR repositories across AWS accounts?** Update the ECR repository policy in Account A to allow Account B's EKS node role to `ecr:GetDownloadUrlForLayer`.
**159. What is a "confused deputy" attack?** Prevented in IRSA by using the `StringEquals` condition checking the `aud` and `sub` (namespace/serviceaccount name) of the OIDC token.
**160. Can an EKS cluster span multiple regions?** No. EKS clusters are strictly regional. Use Route 53 to load balance across multiple regional clusters.

## Storage, Observability & GitOps (161-200)
**161. What is the Amazon EBS CSI Driver?** Allows EKS clusters to manage the lifecycle of Amazon EBS volumes for persistent block storage.
**162. How do you dynamically provision an EBS volume?** Create a `PersistentVolumeClaim` (PVC); the EBS CSI driver automatically provisions the volume and binds it.
**163. What is the default EBS volume type provisioned?** Usually `gp2`, but `gp3` is recommended and can be set in the StorageClass.
**164. Is EBS ReadWriteMany (RWX)?** No, EBS is ReadWriteOnce (RWO). It can only be mounted to a single node at a time.
**165. How do you achieve ReadWriteMany (RWX) in EKS?** Use the Amazon EFS CSI Driver. EFS is an NFS file system that can be mounted by thousands of pods simultaneously.
**166. What is Amazon FSx for Lustre?** A high-performance file system used for machine learning and HPC workloads in EKS via the FSx CSI driver.
**167. How do you resize an EBS volume in EKS?** Modify the `storage` request in the PVC manifest. The EBS CSI driver handles the AWS API call and filesystem expansion.
**168. How do you backup persistent volumes?** Use the EBS CSI driver's snapshotting capabilities, managed via the `VolumeSnapshot` CRD.
**169. What is CloudWatch Container Insights?** Automatically collects, aggregates, and summarizes metrics and logs from containerized apps on EKS.
**170. How is Container Insights deployed?** Usually via the CloudWatch Observability EKS Add-on (installs the CloudWatch Agent).
**171. What is AWS Distro for OpenTelemetry (ADOT)?** AWS's supported distribution of OpenTelemetry for collecting distributed traces and metrics from EKS.
**172. How do you scrape Prometheus metrics in EKS?** Deploy Prometheus, or use Amazon Managed Service for Prometheus (AMP).
**173. What is Amazon Managed Grafana (AMG)?** A fully managed Grafana service that natively queries AMP, CloudWatch, and X-Ray.
**174. How do you forward EKS logs to OpenSearch (Elasticsearch)?** Use Fluent Bit (deployed as a DaemonSet) to tail container logs and output them to an Amazon OpenSearch domain.
**175. Why use Fluent Bit instead of Fluentd?** Fluent Bit is written in C, has a much smaller memory/CPU footprint, and is the AWS recommended log router.
**176. What is AWS X-Ray?** A distributed tracing system used to track requests across microservices.
**177. How does a pod use AWS X-Ray?** Deploy the X-Ray daemon as a sidecar or DaemonSet, and instrument the application code using the X-Ray SDK.
**178. What is GitOps?** Operating infrastructure and applications by keeping their desired state in Git, with automated agents syncing it to the cluster.
**179. What is ArgoCD?** A declarative, GitOps continuous delivery tool for Kubernetes.
**180. What is Flux?** Another popular CNCF GitOps tool. (AWS offers Flux as the engine behind EKS Anywhere and some native AWS GitOps features).
**181. Why is GitOps better than `kubectl apply` in CI/CD?** Better security (CI/CD doesn't need cluster admin credentials), drift detection, and easy rollbacks via `git revert`.
**182. How do you manage EKS infrastructure itself?** Using Terraform (the `terraform-aws-eks` module) or AWS CDK.
**183. What is Helm?** The package manager for K8s; uses templates to render YAML manifests based on input values.
**184. How does ArgoCD handle Helm charts?** ArgoCD can natively inflate Helm charts and sync the resulting manifests to the cluster.
**185. What is Kustomize?** A template-free way to customize K8s manifests using patches/overlays; natively supported by `kubectl` and ArgoCD.
**186. How do you monitor EKS costs?** Use AWS Cost Explorer and Kubecost.
**187. What is Kubecost?** An open-source tool (partnered with AWS) that provides granular visibility into EKS costs per namespace, deployment, and pod.
**188. How does the Horizontal Pod Autoscaler (HPA) work?** Monitors metrics (via Metrics Server) and scales the number of replica pods up/down.
**189. What is the Vertical Pod Autoscaler (VPA)?** Monitors usage and automatically adjusts the CPU/Memory `requests` and `limits` of pods (requires pod restarts).
**190. What is KEDA?** Kubernetes Event-Driven Autoscaling. Scales pods based on external metrics (like SQS queue depth or Kafka lag).
**191. Can you scale pods based on an SQS queue?** Yes, use KEDA with the AWS SQS scaler. It can scale pods to 0 when the queue is empty.
**192. What happens to stateful apps during a node failure?** Pods are rescheduled. If using EBS, the volume is detached from the dead node and reattached to the new node.
**193. Why might an EBS volume fail to attach to a new node?** Multi-Attach is not supported for standard EBS; the volume must fully detach from the dead node first (can take several minutes).
**194. What is Chaos Engineering in EKS?** Deliberately injecting failures (killing nodes, dropping packets) using tools like AWS Fault Injection Simulator (FIS) or Chaos Mesh to test resilience.
**195. What are EKS Pod Templates?** Used by Karpenter to define default configurations for dynamically provisioned nodes.
**196. How do you do Blue/Green deployments in EKS?** Deploy two identical environments (Blue and Green). Update the Ingress/Service selector to point from Blue to Green. Tools like Argo Rollouts automate this.
**197. How do you do Canary deployments?** Send a small percentage of traffic (e.g., 5%) to the new version using an Ingress Controller (like ALB or Nginx) or Service Mesh (App Mesh/Istio), scaling up if no errors occur.
**198. What is the difference between a Readiness Probe and an ALB Health Check?** Readiness Probe tells K8s if the pod is ready to receive traffic. The ALB Health Check tells the AWS Load Balancer if the target is healthy. They should align.
**199. What is AWS Controllers for Kubernetes (ACK)?** Lets you define and use AWS service resources (like S3 buckets, RDS databases) directly from Kubernetes using custom resources (CRDs).
**200. What is Crossplane?** An open-source K8s add-on that enables you to assemble infrastructure from multiple cloud vendors (AWS, GCP) via K8s manifests, essentially turning K8s into a universal control plane.

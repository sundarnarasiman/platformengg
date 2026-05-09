# Amazon EKS Interview Q&A for Principal Platform Engineers

This document contains 50 interview questions and answers focused exclusively on Amazon Elastic Kubernetes Service (EKS), a critical technology for the Appian Platform Engineer role.

## EKS Architecture & Control Plane (1-10)

**1. Describe the architecture of an Amazon EKS cluster.**
*Answer:* EKS consists of two main components: the Control Plane (managed by AWS) and the Data Plane (worker nodes managed by you or AWS). The Control Plane runs the Kubernetes API server, etcd, and scheduler across multiple Availability Zones for high availability. The Data Plane consists of EC2 instances or Fargate profiles that run your application Pods.

**2. How does AWS manage the etcd cluster in EKS?**
*Answer:* AWS fully manages the etcd database for EKS. It runs on dedicated EC2 instances, is spread across three Availability Zones, is encrypted at rest using KMS, and is automatically backed up. If an etcd node fails, AWS automatically replaces it.

**3. What is the difference between EKS public and private endpoint access?**
*Answer:* 
*   *Public:* The Kubernetes API server is accessible from the internet. Worker nodes communicate with the API server over the public internet (or via NAT Gateway).
*   *Private:* The API server is only accessible from within your VPC (via an ENI created by AWS). All communication between nodes and the API server stays on the AWS backbone.
*   *Public and Private:* The API server is accessible from the internet, but worker nodes communicate with it privately. (Recommended for highest security while retaining external tool access).

**4. How do you upgrade an EKS cluster with zero downtime?**
*Answer:* 
1. Upgrade the EKS Control Plane version via the AWS Console/CLI.
2. Upgrade the CoreDNS, kube-proxy, and VPC CNI add-ons.
3. Update the worker nodes. For Managed Node Groups, you trigger a rolling update. For custom nodes, you launch a new Node Group with the new AMI, cordon/drain the old nodes, and terminate them once empty.

**5. What are EKS Add-ons?**
*Answer:* EKS add-ons are operational software that provide critical cluster functionality (like CoreDNS, kube-proxy, VPC CNI, and EBS CSI driver). EKS allows you to manage their lifecycle (install, upgrade) directly through the AWS API/Console rather than manually applying YAML manifests.

**6. What happens to your cluster if the EKS Control Plane goes down?**
*Answer:* AWS provides an SLA for the control plane. If it goes down, you cannot make changes to the cluster (create pods, update deployments, use `kubectl`), but existing Pods running on the Data Plane will continue to serve traffic and communicate with each other.

**7. Can you SSH into the EKS Control Plane nodes?**
*Answer:* No. The Control Plane is fully managed by AWS within an AWS-owned VPC. You do not have OS-level access to the API server or etcd nodes.

**8. How do you monitor the EKS Control Plane logs?**
*Answer:* You can enable EKS Control Plane logging, which sends logs (api, audit, authenticator, controllerManager, scheduler) directly to Amazon CloudWatch Logs.

**9. What is EKS Anywhere?**
*Answer:* It's an extension of EKS that allows you to create and operate Kubernetes clusters on your own on-premises infrastructure (VMware vSphere or bare metal) using the same software (EKS Distro) and operational tooling as cloud-based EKS.

**10. How do you handle disaster recovery for an EKS cluster?**
*Answer:* Ensure infrastructure is codified (Terraform). Store K8s manifests in Git (GitOps/ArgoCD). Back up persistent data (EBS snapshots, RDS backups). For K8s state (if relying on CRDs or cluster-specific configs), use tools like Velero to back up the cluster state to S3. In a disaster, recreate the EKS cluster via Terraform and let ArgoCD sync the workloads.

## Compute & Node Management (11-20)

**11. What are the three ways to run worker nodes in EKS?**
*Answer:* 
1) **Self-Managed Nodes:** You create EC2 instances (usually via ASGs) and manually join them to the cluster.
2) **Managed Node Groups:** AWS automates the provisioning, lifecycle, and updates of EC2 nodes for you.
3) **AWS Fargate:** Serverless compute for containers. You don't manage EC2 instances at all; AWS provisions exact compute per Pod.

**12. What is Karpenter, and how is it better than Cluster Autoscaler?**
*Answer:* Karpenter is an open-source, next-generation node provisioning project built by AWS. Instead of relying on AWS Auto Scaling Groups (like Cluster Autoscaler), Karpenter directly observes pending Pods and provisions right-sized EC2 instances (just-in-time) in seconds, significantly improving scaling speed and cost efficiency.

**13. When would you choose Fargate over Managed Node Groups?**
*Answer:* Choose Fargate when you want zero operational overhead for managing EC2 instances, patching OS, or right-sizing nodes. Avoid Fargate if you need DaemonSets, privileged containers, host networking, or GPUs, as Fargate does not support these.

**14. How does AWS Fargate determine how much CPU and Memory to allocate?**
*Answer:* Fargate looks at the `requests` and `limits` defined in your Pod specification and provisions a compute profile that matches or slightly exceeds those requirements. You are billed based on the provisioned CPU and memory.

**15. What is a Fargate Profile?**
*Answer:* It's a configuration that tells EKS which Pods should run on Fargate. It uses selectors based on Kubernetes Namespaces and labels. If a Pod matches the profile, EKS automatically schedules it on Fargate instead of an EC2 node.

**16. How do you gracefully terminate nodes in a Managed Node Group during an update?**
*Answer:* EKS automatically handles this by cordoning the node (preventing new pods) and draining it (evicting existing pods gracefully based on PodDisruptionBudgets) before terminating the underlying EC2 instance.

**17. What is an EKS AMI, and why is it important?**
*Answer:* Amazon provides EKS-optimized Linux AMIs (based on Amazon Linux 2, Bottlerocket, or Ubuntu). They come pre-configured with Docker/containerd, kubelet, and the AWS IAM Authenticator required to join the cluster.

**18. What is Bottlerocket?**
*Answer:* An open-source, Linux-based operating system purpose-built by AWS for running containers. It has a significantly smaller footprint, faster boot times, and a reduced attack surface compared to general-purpose OSs like Amazon Linux 2.

**19. How do you run Spot Instances safely in EKS?**
*Answer:* Create a separate Managed Node Group configured for Spot capacity. Crucially, install the AWS Node Termination Handler. It monitors EC2 metadata for the 2-minute Spot interruption warning and safely cordons/drains the node before AWS reclaims it.

**20. Explain Taints and Tolerations in the context of EKS Node Groups.**
*Answer:* You can apply Taints to a Node Group (e.g., `gpu=true:NoSchedule`). Only Pods with a matching Toleration can be scheduled on those nodes. This is used to ensure expensive GPU nodes or specific Spot instances are only used by workloads that explicitly request them.

## Networking (21-30)

**21. Explain the Amazon VPC CNI plugin.**
*Answer:* The VPC CNI plugin allows Kubernetes Pods to have the same IP address as they do on the VPC network. It assigns secondary IP addresses from the EC2 instance's Elastic Network Interface (ENI) directly to the Pods.

**22. What is the biggest challenge with the VPC CNI plugin, and how do you solve it?**
*Answer:* **IP Exhaustion.** Because every Pod gets a VPC IP address, dense clusters can quickly exhaust the subnet's IP space. 
*Solution:* Enable Custom Networking (assigning Pods to a different subnet, like a /16 secondary CIDR) or enable Prefix Delegation (assigning /28 IP prefixes to ENIs instead of single IPs, vastly increasing pod density per node).

**23. How do you expose an application running in EKS to the internet?**
*Answer:* Create an Ingress resource and use the AWS Load Balancer Controller. It will automatically provision an AWS Application Load Balancer (ALB), configure listener rules, and route internet traffic directly to the Pod IPs.

**24. What is the AWS Load Balancer Controller?**
*Answer:* It's a Kubernetes controller that manages AWS Elastic Load Balancers. It satisfies Kubernetes `Ingress` resources by provisioning Application Load Balancers (ALBs) and satisfies `Service` of type `LoadBalancer` by provisioning Network Load Balancers (NLBs).

**25. What is "Security Groups for Pods"?**
*Answer:* Normally, a Security Group is attached to the EC2 worker node, meaning all Pods on that node share the same network permissions. "Security Groups for Pods" integrates with the VPC CNI to attach an AWS Security Group directly to an individual Pod, allowing strict, pod-level network isolation.

**26. How do you handle internal service-to-service communication across different EKS clusters?**
*Answer:* Use a Service Mesh (like AWS App Mesh or Istio), or use the AWS Cloud Map controller. Alternatively, if clusters share a VPC or are peered, an internal NLB or ALB can bridge them.

**27. What happens if a Pod's IP address needs to communicate with an external internet service?**
*Answer:* If the Pod is in a private subnet, the traffic routes through the VPC CNI to the EC2 host, out the host's ENI, follows the VPC route table to a NAT Gateway in a public subnet, and then out the Internet Gateway (IGW).

**28. Can you use alternate CNIs (like Calico or Cilium) with EKS?**
*Answer:* Yes. While VPC CNI is the default and provides native AWS networking, you can install Calico for strict Network Policies or replace VPC CNI entirely with Cilium (via eBPF) for advanced networking, observability, and avoiding VPC IP exhaustion.

**29. What is the difference between Target Type `instance` and `ip` in the AWS Load Balancer Controller?**
*Answer:* 
*   `instance`: The ALB routes traffic to the NodePort on the EC2 instances, and kube-proxy routes it to the Pod.
*   `ip` (Recommended): The ALB routes traffic directly to the Pod's IP address within the VPC, bypassing kube-proxy. This is more efficient and reduces network hops.

**30. How do you implement Kubernetes Network Policies in EKS?**
*Answer:* The AWS VPC CNI does not support Network Policies natively by default. You must install a network policy engine, such as the Calico Network Policy provider, to enforce isolation between pods within the cluster. (Note: AWS recently introduced native Network Policy support for VPC CNI, but Calico remains heavily used).

## Security & Identity (31-40)

**31. How does authentication work in EKS?**
*Answer:* EKS uses the `aws-iam-authenticator`. When you run `kubectl`, it generates a short-lived token using your AWS IAM credentials. The EKS API server receives this token, verifies it with AWS IAM, and maps the IAM User/Role to a Kubernetes RBAC User/Group via the `aws-auth` ConfigMap.

**32. What is the `aws-auth` ConfigMap?**
*Answer:* It's a ConfigMap in the `kube-system` namespace that maps AWS IAM Roles or IAM Users to Kubernetes RBAC users or groups (e.g., mapping an IAM Role `DevTeam` to the K8s group `system:masters`).

**33. What is IRSA (IAM Roles for Service Accounts)?**
*Answer:* IRSA allows you to assign an AWS IAM Role to a Kubernetes Service Account. Pods using that Service Account can then securely assume the IAM Role to access AWS services (like S3 or DynamoDB). This implements Least Privilege, replacing the insecure practice of assigning broad IAM permissions to the underlying EC2 node.

**34. How does IRSA work under the hood?**
*Answer:* It uses OpenID Connect (OIDC). You create an OIDC identity provider in AWS for your EKS cluster. When a Pod starts, EKS injects a projected service account token (a JWT) into the Pod. The AWS SDK in the Pod exchanges this JWT with AWS STS (via `AssumeRoleWithWebIdentity`) for temporary AWS credentials.

**35. How do you encrypt Kubernetes secrets at rest in EKS?**
*Answer:* By default, secrets are base64 encoded and stored in etcd (which is encrypted at the disk level by AWS). For true application-level encryption, you enable KMS Envelope Encryption during cluster creation. This uses an AWS KMS key to encrypt the secrets before they are written to etcd.

**36. How do you pull container images securely from a private ECR repository?**
*Answer:* Ensure the EC2 Node IAM Role (or the Pod via IRSA) has the `AmazonEC2ContainerRegistryReadOnly` managed policy. EKS/kubelet handles the ECR authentication automatically.

**37. What is OPA Gatekeeper or Kyverno, and why use it with EKS?**
*Answer:* They are Policy-as-Code admission controllers. They enforce organizational standards before resources are created in the cluster. For example, blocking Pods that run as `root`, requiring all images to come from a private ECR registry, or enforcing mandatory labels.

**38. If an IAM user creates an EKS cluster, who has access to it?**
*Answer:* Only the specific IAM user or role that created the cluster has `system:masters` (full admin) permissions by default. This creator is not listed in the `aws-auth` ConfigMap. They must explicitly add other IAM users/roles to `aws-auth` to grant access.

**39. How do you securely manage sensitive application data (like DB passwords) in EKS?**
*Answer:* Do not use Kubernetes Secrets directly. Use the AWS Secrets and Configuration Provider (ASCP) for the Kubernetes Secrets Store CSI Driver. This allows Pods to mount secrets from AWS Secrets Manager or Parameter Store dynamically as volumes, keeping them out of etcd.

**40. What is Amazon EKS Pod Identity?**
*Answer:* Introduced recently, EKS Pod Identity is a simplified alternative to IRSA. It removes the need to configure OIDC providers and manage complex trust relationships manually. You simply link an IAM role to a K8s Service Account using the EKS console/API, and an EKS agent handles the credential delivery.

## Storage, Monitoring & Operations (41-50)

**41. How do you attach a persistent block volume to an EKS Pod?**
*Answer:* Use the Amazon EBS CSI (Container Storage Interface) driver. You define a StorageClass, create a PersistentVolumeClaim (PVC), and the EBS CSI driver dynamically provisions an EBS volume and attaches it to the EC2 instance running the Pod.

**42. Can two Pods in EKS write to the same EBS volume simultaneously?**
*Answer:* No. EBS is ReadWriteOnce (RWO). To allow multiple Pods across different nodes to read and write to the same storage simultaneously (ReadWriteMany), you must use Amazon EFS (Elastic File System) via the EFS CSI driver.

**43. How do you collect and forward container logs to CloudWatch in EKS?**
*Answer:* Deploy Fluent Bit (recommended by AWS) or Fluentd as a DaemonSet. It tails the `/var/log/containers/` directory on each node and forwards the logs to Amazon CloudWatch Logs (or Elasticsearch/OpenSearch).

**44. What is Container Insights?**
*Answer:* A feature of CloudWatch that automatically collects, aggregates, and summarizes metrics and logs from containerized applications and microservices running on EKS. It provides pre-built dashboards for cluster, node, and pod-level CPU/Memory/Network usage.

**45. How do you backup and restore an EKS application?**
*Answer:* Use an open-source tool like Velero. Velero backs up Kubernetes objects (Deployments, Services) to an S3 bucket and triggers AWS API calls to take snapshots of the underlying EBS persistent volumes.

**46. Your Pod is stuck in `Pending` state. How do you troubleshoot?**
*Answer:* 
1. Run `kubectl describe pod <pod-name>`. Look at the Events section.
2. Common causes: Insufficient CPU/Memory on nodes, no node matches the NodeSelector/Affinity rules, a required PVC cannot be provisioned, or the cluster has exhausted IP addresses (VPC CNI limits).

**47. Your Pod is in `CrashLoopBackOff`. How do you troubleshoot?**
*Answer:* 
1. Run `kubectl logs <pod-name>` (or `kubectl logs <pod-name> --previous` if it restarted) to see application exceptions.
2. Run `kubectl describe pod` to check for Liveness probe failures, missing environment variables, or incorrect command/args in the container spec.

**48. How do you implement autoscaling for Pods in EKS?**
*Answer:* Use the Horizontal Pod Autoscaler (HPA). It automatically scales the number of Pods in a Deployment up or down based on observed CPU utilization or custom metrics (like SQS queue length via KEDA).

**49. What is KEDA, and how does it integrate with EKS?**
*Answer:* KEDA (Kubernetes-based Event Driven Autoscaler) allows you to autoscale Pods based on external events. In AWS, KEDA can scale your EKS pods based on the depth of an Amazon SQS queue, the number of messages in a Kafka topic, or CloudWatch metrics, scaling down to zero when idle.

**50. What is Amazon EMR on EKS?**
*Answer:* It's a deployment option that allows you to run Apache Spark workloads on Amazon EKS. Instead of provisioning dedicated EMR clusters (EC2 instances), you submit Spark jobs directly to EKS, sharing the compute pool with your other microservices and improving resource utilization.

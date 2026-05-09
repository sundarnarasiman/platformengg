# 150 Kubernetes Interview Questions & Answers

## Core Concepts (1-20)
**1. What is a Pod?** The smallest deployable computing unit in K8s, representing a single instance of a running process, containing one or more containers.
**2. Difference between a Deployment and a StatefulSet?** Deployments are for stateless applications (interchangeable pods). StatefulSets guarantee ordering, uniqueness, and sticky identity for stateful apps.
**3. What is a DaemonSet?** Ensures a copy of a specific Pod runs on all (or selected) Nodes. Useful for logging agents (Fluentd) or monitoring (Prometheus Node Exporter).
**4. What is a ReplicaSet?** Ensures a specified number of pod replicas are running at any given time. Deployments manage ReplicaSets.
**5. What is a Job?** Creates one or more Pods and ensures that a specified number of them successfully terminate (run to completion).
**6. What is a CronJob?** Manages time-based Jobs, running them on a schedule (like Linux cron).
**7. What is a Namespace?** A virtual cluster within a physical K8s cluster, used to divide cluster resources between multiple users or environments.
**8. Can Pods in different namespaces communicate?** Yes, by default. Namespaces provide logical isolation, not network isolation (unless NetworkPolicies are applied).
**9. What is an Init Container?** A container that runs and completes before the main app containers start. Used for setup scripts or delaying startup.
**10. What is a Sidecar pattern?** A helper container running in the same Pod as the main container (e.g., a logging proxy or service mesh proxy like Envoy).
**11. What is the Pause container?** An infrastructure container in every Pod that holds the network namespace and IP address for the Pod.
**12. How does a Deployment perform a rolling update?** It creates a new ReplicaSet, scales it up gradually while scaling down the old ReplicaSet, ensuring zero downtime.
**13. What is a PodDisruptionBudget (PDB)?** Defines the minimum available (or maximum unavailable) pods during voluntary disruptions (like node drains).
**14. What are Labels and Selectors?** Labels are key/value pairs attached to objects. Selectors are used to query and group objects based on those labels.
**15. What are Annotations?** Key/value pairs attached to objects for non-identifying metadata (not used by selectors), often used by external tools/controllers.
**16. What is a ConfigMap?** An API object used to store non-confidential data in key-value pairs, which can be consumed by Pods as env vars or volumes.
**17. What is a Secret?** Similar to a ConfigMap but for confidential data (passwords, tokens). Stored as base64 encoded strings.
**18. How do you trigger an immediate restart of a Deployment?** `kubectl rollout restart deployment <name>`.
**19. What is the difference between a Pod and a Container?** A Container wraps the application code. A Pod is the K8s wrapper around one or more containers that share network and storage.
**20. What is the `kubectl apply` vs `kubectl create` difference?** `apply` is declarative (creates or updates based on the YAML). `create` is imperative (errors if the object already exists).

## Networking (21-40)
**21. What is a ClusterIP Service?** The default service type. Exposes the service on a cluster-internal IP, reachable only from within the cluster.
**22. What is a NodePort Service?** Exposes the service on each Node's IP at a static port (between 30000-32767). Reachable externally.
**23. What is a LoadBalancer Service?** Provisions an external load balancer from the cloud provider (e.g., AWS ALB/NLB) and routes traffic to the NodePort.
**24. What is an ExternalName Service?** Maps a Service to a DNS name (e.g., `foo.bar.com`) instead of a selector, returning a CNAME record.
**25. What is a Headless Service?** A service with `clusterIP: None`. It doesn't load balance; instead, DNS queries return the IPs of all the backing Pods directly. Useful for StatefulSets.
**26. What is an Ingress?** An API object that manages external access to services in a cluster, providing HTTP/HTTPS routing, load balancing, SSL termination, and name-based virtual hosting.
**27. What is an Ingress Controller?** A daemon running in the cluster (e.g., NGINX Ingress, AWS ALB Controller) that reads Ingress objects and implements the actual routing rules.
**28. How does Kube-Proxy work?** It runs on every node, maintaining network rules (iptables or IPVS) to forward traffic addressed to Services to the backend Pods.
**29. What is CoreDNS?** The default DNS server in K8s. It watches the API server for new Services and creates DNS records for them.
**30. What is the DNS format for a Kubernetes Service?** `<service-name>.<namespace>.svc.cluster.local`.
**31. What is a CNI plugin?** Container Network Interface. It's the standard for configuring network interfaces for Linux containers. Examples: Calico, Flannel, AWS VPC CNI.
**32. What is the Pod network model in K8s?** Every Pod gets its own IP address. Pods can communicate with all other Pods without NAT. Agents on a node can communicate with all Pods on that node.
**33. What is a NetworkPolicy?** An API object that controls the traffic allowed to and from Pods (like a firewall for Pods). Requires a CNI that supports it (like Calico).
**34. Default NetworkPolicy behavior?** If no policies exist in a namespace, all traffic is allowed (default allow). Once a policy selects a pod, default deny is applied for unallowed traffic.
**35. What is SNAT and when does K8s use it?** Source Network Address Translation. Used when a Pod communicates with an external IP outside the cluster; the Node translates the Pod IP to the Node IP.
**36. How do you debug DNS issues in a Pod?** Exec into the pod and run `nslookup kubernetes.default` or check the `/etc/resolv.conf`.
**37. What is Endpoints/EndpointSlice?** Objects that maintain the list of active IP addresses (Pod IPs) that match a Service's selector.
**38. What is the Gateway API?** The next-generation evolution of Ingress, providing more expressive, extensible, and role-oriented routing APIs.
**39. Can a Pod have multiple IP addresses?** Yes, using tools like Multus CNI, a Pod can attach to multiple networks and have multiple interfaces/IPs.
**40. What port does kubelet use?** 10250.

## Storage (41-60)
**41. What is a PersistentVolume (PV)?** A piece of storage in the cluster provisioned by an administrator or dynamically via a StorageClass.
**42. What is a PersistentVolumeClaim (PVC)?** A request for storage by a user. It binds to a PV that matches its size and access mode requests.
**43. What is a StorageClass?** Allows administrators to describe the "classes" of storage they offer (e.g., fast SSD, slow HDD). Used for dynamic volume provisioning.
**44. What are the PV Access Modes?** ReadWriteOnce (RWO - single node), ReadOnlyMany (ROX - many nodes read), ReadWriteMany (RWX - many nodes read/write).
**45. What happens if a PVC requests ReadWriteMany but the provisioner only supports ReadWriteOnce?** The PVC will remain in a "Pending" state and will not bind.
**46. What is the CSI (Container Storage Interface)?** A standard that exposes arbitrary block and file storage systems to containerized workloads, decoupling storage drivers from the core K8s code.
**47. What is an emptyDir volume?** A temporary directory that shares a Pod's lifetime. If the Pod is deleted, the data is lost. Good for scratch space.
**48. What is a hostPath volume?** Mounts a file or directory from the host node's filesystem into a Pod. (Security risk, use carefully).
**49. How do you resize a PVC?** Modify the `storage` request in the PVC manifest. The StorageClass must have `allowVolumeExpansion: true`.
**50. What is a Reclaim Policy?** Determines what happens to the PV when the PVC is deleted. Options: Retain, Recycle (deprecated), Delete.
**51. What is Volume Binding Mode?** `Immediate` binds/provisions storage as soon as the PVC is created. `WaitForFirstConsumer` delays it until a Pod using the PVC is scheduled to a node.
**52. Why is `WaitForFirstConsumer` important for multi-AZ clusters?** It ensures the volume is provisioned in the same Availability Zone where the Pod is eventually scheduled.
**53. How do StatefulSets use PVCs?** Through `volumeClaimTemplates`, generating a unique PVC for each Pod replica (e.g., `data-web-0`, `data-web-1`).
**54. What is a local persistent volume?** Represents a mounted local storage device (disk, partition) directly on a node. Unlike hostPath, K8s understands node affinity for local PVs.
**55. How do you backup Kubernetes persistent data?** Use tools like Velero, which snapshot the underlying cloud provider volumes and backup K8s objects to S3.
**56. Can you mount a ConfigMap as a volume?** Yes, the key-value pairs become files in the mounted directory.
**57. What is the `subPath` property in volume mounts?** It allows you to mount a specific file or folder from a volume rather than the entire volume.
**58. What is Ephemeral Storage?** Storage local to the node used by Pods for scratch space, caching, and logs (managed by `requests` and `limits` just like CPU/RAM).
**59. What happens to EBS volumes if you delete a Deployment?** Nothing, if they are PVs with Retain policy. If they are PVCs dynamically provisioned with Delete policy, and you delete the PVC, the volume is deleted.
**60. What is a Projected Volume?** Maps several existing volume sources (Secrets, ConfigMaps, DownwardAPI) into a single directory.

## Security (61-80)
**61. What is Role-Based Access Control (RBAC)?** Regulates access to K8s API resources based on the roles of individual users within an organization.
**62. Difference between a Role and a ClusterRole?** A Role grants permissions within a specific Namespace. A ClusterRole grants cluster-wide permissions.
**63. What is a RoleBinding vs ClusterRoleBinding?** RoleBinding applies a Role/ClusterRole to users within a Namespace. ClusterRoleBinding applies a ClusterRole globally across all namespaces.
**64. What is a ServiceAccount?** Provides an identity for processes that run in a Pod, allowing them to authenticate to the K8s API.
**65. How does a Pod use a ServiceAccount?** The API server automatically mounts the ServiceAccount token into the Pod at `/var/run/secrets/kubernetes.io/serviceaccount`.
**66. What is a SecurityContext?** Defines privilege and access control settings for a Pod or Container (e.g., `runAsUser`, `readOnlyRootFilesystem`).
**67. What is `privileged: true` in a SecurityContext?** It grants the container all root capabilities on the host node, bypassing container isolation. Highly dangerous.
**68. How do you prevent a container from running as root?** Set `runAsNonRoot: true` in the Pod's SecurityContext.
**69. What are Pod Security Standards (PSS) / Pod Security Admission (PSA)?** The built-in replacement for PodSecurityPolicies (PSP). Enforces isolation levels (Privileged, Baseline, Restricted) via namespace labels.
**70. What is an Admission Controller?** A piece of code that intercepts requests to the API server prior to persistence but after authentication/authorization.
**71. Difference between Mutating and Validating Webhooks?** Mutating webhooks can modify the request object (e.g., inject a sidecar). Validating webhooks can only accept or reject the request.
**72. What is OPA Gatekeeper / Kyverno?** Open-source Policy-as-Code engines implemented as admission controllers to enforce custom governance rules on the cluster.
**73. How are K8s Secrets stored internally?** By default, they are stored as plain text (base64 encoded) in etcd.
**74. How do you secure etcd data?** Enable encryption at rest in the K8s API server using an EncryptionConfiguration (often backed by a cloud KMS).
**75. What is the Kubelet TLS Bootstrap process?** A secure way for new worker nodes to generate a private key and request a signed certificate from the API server to join the cluster.
**76. Can you restrict egress traffic to specific external domains?** Native K8s NetworkPolicies only support IP CIDRs. To restrict by domain (FQDN), you need advanced CNIs (Cilium/Calico Enterprise) or a service mesh (Istio).
**77. What is image pull secret?** A Secret containing docker registry credentials used by kubelet to pull private container images.
**78. What does `automountServiceAccountToken: false` do?** Prevents Kubernetes from automatically injecting the API token into a Pod, increasing security for apps that don't need API access.
**79. How do you audit Kubernetes API requests?** Enable K8s Audit Logging by passing an audit policy file to the kube-apiserver, recording who did what, when.
**80. What is dropping Linux Capabilities?** Removing specific root privileges from a container (e.g., `drop: ["ALL"]` in SecurityContext) to enforce least privilege.

## Scheduling (81-100)
**81. What is the kube-scheduler?** The control plane component responsible for assigning newly created Pods to available worker Nodes based on resource requirements and constraints.
**82. What are Taints and Tolerations?** Taints are applied to Nodes to repel Pods. Tolerations are applied to Pods to allow them to schedule on tainted nodes.
**83. What is the `NoExecute` taint effect?** It evicts any already-running Pods on the node that do not have a matching toleration.
**84. What is `nodeSelector`?** The simplest form of node constraint. A Pod will only schedule on nodes with labels matching the `nodeSelector` map.
**85. What is Node Affinity?** A more expressive version of nodeSelector supporting soft ("prefer") and hard ("require") scheduling rules, and operators like `In`, `NotIn`, `Exists`.
**86. What is Pod Affinity and Anti-Affinity?** Rules that constrain a Pod to be scheduled on (or away from) a node based on the labels of *other Pods* already running on that node.
**87. Use case for Pod Anti-Affinity?** To ensure high availability by spreading replicas of a web server across different nodes or availability zones (using `topologyKey`).
**88. What are CPU/Memory Requests?** The guaranteed amount of resources reserved for a container. The scheduler uses requests to find a node with enough capacity.
**89. What are CPU/Memory Limits?** The hard cap on resources a container can use. If it exceeds the memory limit, it is OOMKilled. Exceeding CPU limit results in throttling.
**90. What is a LimitRange?** An API object that sets default requests/limits and min/max constraints for Pods and PVCs within a namespace.
**91. What is a ResourceQuota?** An API object that restricts the total aggregate resource consumption (total CPU, total RAM, total number of Pods) allowed in a namespace.
**92. How does the scheduler handle a Pod with requests larger than any node?** The Pod remains in a `Pending` state forever.
**93. What is the Descheduler?** An optional add-on that evicts Pods from nodes based on policies (e.g., to rebalance a cluster after new nodes are added).
**94. What are PriorityClasses?** They define the importance of a Pod. If a node is full, the scheduler can preempt (evict) lower priority Pods to make room for higher priority ones.
**95. What is `topologySpreadConstraints`?** Allows you to control how Pods are spread across your cluster among failure-domains such as regions, zones, nodes.
**96. What does `cordon` do to a node?** Marks it as unschedulable. No new Pods will be assigned to it, but existing Pods keep running.
**97. What does `drain` do to a node?** Cordons the node and gracefully evicts all existing Pods, respecting PodDisruptionBudgets. Used for node maintenance.
**98. What is the Kubelet's role in scheduling?** Kubelet doesn't schedule; it watches the API server for Pods assigned to its node and ensures their containers are running.
**99. Can you run multiple schedulers?** Yes, you can deploy a custom scheduler and specify `schedulerName` in a Pod spec to use it instead of the default.
**100. How does Cluster Autoscaler work?** It watches for Pods stuck in `Pending` due to insufficient resources and triggers the cloud provider to add more nodes.

## Architecture & Internals (101-120)
**101. What is the kube-apiserver?** The brain of K8s. It validates and configures data for API objects and is the only component that communicates with etcd.
**102. What is etcd?** A highly-available, distributed key-value store used as Kubernetes' backing store for all cluster data.
**103. What is the kube-controller-manager?** Runs controller processes (Node controller, ReplicaSet controller, Endpoint controller) that regulate the state of the cluster.
**104. What is the cloud-controller-manager?** Embeds cloud-specific control logic (like creating load balancers or managing cloud volumes), separating it from core K8s code.
**105. What is the Kubelet?** The primary "node agent" that runs on each node. It ensures containers are running in a Pod according to the PodSpec.
**106. How do the components communicate?** All components (scheduler, controller-manager, kubelet, kube-proxy) communicate exclusively via the kube-apiserver. They do not talk to each other directly.
**107. What happens if etcd loses quorum?** The cluster becomes read-only. You cannot deploy new workloads or update state until quorum is restored.
**108. What is the Container Runtime?** The software responsible for running containers (e.g., containerd, CRI-O). K8s communicates with it via the Container Runtime Interface (CRI).
**109. What is a "Static Pod"?** A Pod managed directly by the kubelet on a specific node, without the API server observing it. Used for bootstrapping control plane components.
**110. Where are Static Pods configured?** By placing a YAML file in a specific directory on the host (usually `/etc/kubernetes/manifests`), monitored by kubelet.
**111. What is the declarative model of K8s?** You declare the "desired state" (via YAML), and controllers continuously run reconciliation loops to make the "actual state" match the desired state.
**112. Explain the Level-Triggered vs Edge-Triggered concept in K8s.** K8s controllers are level-triggered; they look at the current *state* of the system and act, rather than relying on catching every *event* (edge-triggered).
**113. What is garbage collection in K8s?** A mechanism that automatically deletes dependent objects when their owner is deleted (e.g., deleting a ReplicaSet deletes its Pods).
**114. What are OwnerReferences?** A field in metadata that links dependent objects to their parent object, utilized by the garbage collector.
**115. How does Kubelet monitor container health?** Using probes (Liveness, Readiness, Startup) defined in the PodSpec.
**116. What is the purpose of finalizers?** They are keys in metadata that tell Kubernetes to wait until specific conditions are met (or cleanup logic is executed) before fully deleting an object.
**117. What is `kubectl proxy`?** Opens a connection to the API server over HTTP on localhost, authenticating automatically using your kubeconfig credentials.
**118. What is `kubectl port-forward`?** Forwards local ports to a port on a specific Pod, circumventing services/ingress for debugging.
**119. What is a Kubeconfig file?** A YAML file containing cluster server URLs, CA certificates, and user authentication credentials used by `kubectl`.
**120. How is Kubernetes versioned?** Major.Minor.Patch (e.g., 1.28.3). Minor versions are released roughly every 4 months.

## Troubleshooting & Observability (121-140)
**121. A Pod is in `CrashLoopBackOff`. How do you troubleshoot?** Use `kubectl logs <pod>` and `kubectl describe pod <pod>`. Look for app errors, missing env vars, or OOMKilled events.
**122. A Pod is stuck in `Pending`. Why?** Usually insufficient node resources (CPU/RAM), unsatisfied node affinity/selectors, taints without tolerations, or PVC provisioning failures.
**123. A Pod is in `ImagePullBackOff`. Why?** Invalid image name/tag, private registry requiring ImagePullSecrets, or the node lacks network access to the registry.
**124. A Service is not routing traffic to a Pod. What to check?** Ensure the Service `selector` exactly matches the Pod `labels`. Check `kubectl get endpoints <service-name>` to see if IPs are mapped.
**125. What does `OOMKilled` mean?** Out Of Memory. The container exceeded its configured Memory Limit and the kernel terminated the process.
**126. What is the Metrics Server?** A cluster-wide aggregator of resource usage data. Required for the Horizontal Pod Autoscaler (HPA) and `kubectl top` to work.
**127. Difference between Liveness and Readiness probes?** Liveness checks if the app is alive; if it fails, kubelet restarts the container. Readiness checks if it can serve traffic; if it fails, the pod is removed from Service endpoints.
**128. What is a Startup probe?** Verifies if the application within the container has started. Disables liveness/readiness checks until it succeeds. Useful for slow-starting legacy apps.
**129. How do you view previous logs if a container restarted?** `kubectl logs <pod-name> --previous`.
**130. What is `kubectl debug`?** Creates an ephemeral container attached to a running Pod for debugging purposes, useful when the main container lacks a shell.
**131. How do you find which node a pod is running on?** `kubectl get pods -o wide`.
**132. What happens if a worker node crashes?** The API server marks the node as `NotReady`. After 5 minutes (default `pod-eviction-timeout`), it evicts the Pods, and ReplicaSets spin up new ones on healthy nodes.
**133. How do you monitor K8s cluster health?** Typically using Prometheus (for metric collection) and Grafana (for visualization), often deployed via kube-prometheus-stack.
**134. What is cAdvisor?** An open-source agent integrated into kubelet that monitors and gathers resource usage and performance metrics of running containers.
**135. What is a "Split Brain" scenario in etcd?** When a network partition occurs and the cluster splits into two groups that both think they are the leader. Etcd prevents this by requiring a strict majority (quorum) to write data.
**136. Why is my Ingress returning a 502 Bad Gateway?** The Ingress Controller cannot communicate with the backend Service. Check if the Pods are Ready, endpoints exist, and ports match.
**137. How to run a temporary pod to test network connectivity?** `kubectl run busybox --image=busybox --rm -it -- sh`.
**138. How do you stream logs from multiple pods belonging to a deployment?** `kubectl logs -f deployment/<name>`.
**139. What is Kube-state-metrics?** A service that listens to the API server and generates metrics about the state of the objects (e.g., number of ready replicas, pod phase).
**140. How to temporarily prevent traffic from hitting a specific pod for debugging?** Change or remove the label on the pod that the Service selector relies on.

## Advanced & CRDs (141-150)
**141. What is a Custom Resource Definition (CRD)?** An extension of the Kubernetes API that allows you to define custom objects (e.g., `Prometheus`, `Certificate`).
**142. What is the Operator Pattern?** Software that encodes domain-specific operational knowledge (e.g., how to backup a database) into a custom controller that watches CRDs.
**143. How does Helm differ from Kustomize?** Helm is a package manager that uses Go templating to inject values. Kustomize is a configuration management tool built into kubectl that uses a patching/overlay model without templates.
**144. What is GitOps?** A practice where a Git repository is the single source of truth for the cluster state. Tools like ArgoCD or Flux automatically sync K8s with Git.
**145. What is a Service Mesh?** A dedicated infrastructure layer (like Istio or Linkerd) for facilitating service-to-service communications, providing mTLS, tracing, and advanced routing via sidecar proxies.
**146. How do you handle certificate management in K8s?** Using `cert-manager`, an operator that automates the issuance and renewal of TLS certificates from Let's Encrypt or private CAs.
**147. What is API Server Aggregation?** Allows you to extend K8s with custom APIs that feel like core APIs but are served by a separate custom API server registered with the main kube-apiserver.
**148. What is Vertical Pod Autoscaler (VPA)?** Automatically adjusts the CPU and Memory *requests and limits* for Pods based on historical usage, instead of adding more pod replicas.
**149. Why shouldn't you use HPA and VPA on CPU/Memory simultaneously?** They will conflict. VPA will try to resize the pod, causing a restart, while HPA will try to scale replicas based on the same metric, leading to erratic behavior.
**150. What is eBPF and how does it relate to Kubernetes?** Extended Berkeley Packet Filter allows running sandboxed programs in the Linux kernel without changing kernel source code. Used by modern CNIs like Cilium for ultra-fast networking, security, and observability without using iptables.

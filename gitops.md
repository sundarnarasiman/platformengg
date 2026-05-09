# 50 GitOps Interview Questions & Answers

## GitOps Fundamentals & Principles (1-10)

**1. What is GitOps?**
*Answer:* An operational framework that takes DevOps best practices used for application development (like version control, code review, and CI/CD pipelines) and applies them to infrastructure automation. Git serves as the single source of truth.

**2. What are the four core principles of GitOps?**
*Answer:* 1) The entire system is described declaratively. 2) The canonical desired system state is versioned in Git. 3) Approved changes can be automatically applied to the system. 4) Software agents ensure correctness and alert on divergence (drift).

**3. How does GitOps differ from traditional CI/CD?**
*Answer:* Traditional CI/CD uses a "Push" model where the CI server (like Jenkins) runs `kubectl apply` or `terraform apply`. GitOps uses a "Pull" model where an agent inside the cluster (like ArgoCD) monitors Git and pulls changes, meaning the CI server doesn't need admin access to the cluster.

**4. What is the concept of "Drift" in GitOps?**
*Answer:* Drift occurs when the actual state of the infrastructure (e.g., in a Kubernetes cluster) diverges from the desired state declared in the Git repository (e.g., someone manually edits a Deployment via `kubectl edit`).

**5. How does GitOps handle drift?**
*Answer:* The GitOps agent continuously monitors the cluster state against the Git state. If drift is detected, it can either automatically revert the manual changes (self-healing) or generate an alert for an operator to investigate.

**6. Why is immutability important in GitOps?**
*Answer:* Immutability ensures that once a change is committed and applied, the underlying infrastructure is not manually patched. Instead, new commits replace old states, ensuring predictable, reproducible environments and easy rollbacks.

**7. Can GitOps be used for infrastructure outside of Kubernetes?**
*Answer:* Yes. While popularized by Kubernetes, GitOps principles can apply to cloud infrastructure (via tools like Crossplane or Terraform/Atlantis) or server configuration, provided the state can be defined declaratively.

**8. What is the role of Continuous Integration (CI) in a GitOps workflow?**
*Answer:* CI is responsible for testing the code, building the container image, running security scans, and pushing the image to a registry. It then commits the new image tag to the infrastructure/manifest Git repository, triggering the GitOps (CD) pull process.

**9. Why separate the application code repository from the infrastructure/manifest repository?**
*Answer:* Separation of concerns. It prevents CI loops (changing a manifest shouldn't trigger an app rebuild), allows different access controls (devs own app code, platform owns manifests), and provides a clear audit trail of deployment changes separate from code changes.

**10. What is a declarative system, and why is it necessary for GitOps?**
*Answer:* A declarative system describes *what* the final state should look like (e.g., "I want 3 web servers"), rather than imperative scripts dictating *how* to get there (e.g., "Launch server 1, launch server 2"). It is required because the GitOps agent needs a target state to continuously reconcile against.

## Architecture & Workflows (11-20)

**11. Explain the "Pull-based" deployment model.**
*Answer:* The deployment agent runs *inside* the target environment (e.g., the Kubernetes cluster). It continuously polls the Git repository for changes. When a change is detected, the agent pulls the manifests and applies them locally. 

**12. What is the primary security advantage of the Pull model over the Push model?**
*Answer:* In a Push model, the external CI/CD server needs highly privileged credentials (cluster admin) to push changes. If the CI server is compromised, the cluster is compromised. In a Pull model, the cluster only needs read access to the Git repo; no inbound cluster access is required.

**13. Describe a typical GitOps pipeline for a microservice.**
*Answer:* Developer pushes code -> CI runs tests and builds a Docker image -> CI pushes image to ECR -> CI updates the image tag in the `env/prod/deployment.yaml` in the manifest Git repo -> ArgoCD detects the commit -> ArgoCD syncs the cluster state to match Git.

**14. How do you handle environment promotion (e.g., Dev -> Staging -> Prod) in GitOps?**
*Answer:* Typically done using Git branching strategies (e.g., merging the `staging` branch into `main`/`prod`) or directory structures (e.g., copying/patching manifests from a `staging/` folder to a `prod/` folder) which the GitOps agent monitors.

**15. What is Kustomize, and how is it used in GitOps?**
*Answer:* A tool to customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched. It uses a `base` and `overlays` (like dev, prod) model. GitOps agents natively support rendering Kustomize before applying.

**16. How does Helm fit into a GitOps workflow?**
*Answer:* While Helm is an imperative package manager, GitOps tools like ArgoCD can declaratively monitor a Helm chart repository (or Git repo containing `values.yaml` overrides) and render the Helm templates into raw Kubernetes manifests before applying them to the cluster.

**17. What is a "Reconciliation Loop"?**
*Answer:* The continuous process where a GitOps controller compares the desired state (in Git) with the actual state (in the cluster) and takes action (creates, updates, or deletes resources) to make the actual state match the desired state.

**18. How do you perform a rollback in a GitOps workflow?**
*Answer:* You simply use `git revert` to revert the commit that introduced the bad change in the manifest repository. The GitOps agent detects the reverted state and syncs the cluster back to the previous stable state.

**19. What happens if the Git repository goes down?**
*Answer:* The GitOps agent cannot pull new changes, so deployments are paused. However, the cluster continues to run its current state perfectly fine. Once Git is back online, reconciliation resumes automatically.

**20. Can you use GitOps to manage the GitOps tool itself?**
*Answer:* Yes, this is called the "App of Apps" pattern or self-management. The GitOps tool (like ArgoCD) monitors a Git repository that contains the manifests for its own configuration, allowing it to update itself declaratively.

## GitOps Tools (ArgoCD, Flux, Crossplane) (21-30)

**21. What is ArgoCD?**
*Answer:* A declarative, GitOps continuous delivery tool for Kubernetes, known for its powerful UI, SSO integration, and extensive feature set for managing application states across multiple clusters.

**22. What is an ArgoCD `Application` Custom Resource (CR)?**
*Answer:* A CRD that defines a source (Git repository, path, target revision) and a destination (Kubernetes cluster URL and namespace). ArgoCD uses this to know what to sync and where.

**23. What is the "App of Apps" pattern in ArgoCD?**
*Answer:* An ArgoCD `Application` that, instead of pointing to raw deployment manifests, points to a repository containing *other* `Application` manifests. This allows you to bootstrap an entire cluster (monitoring, logging, apps) from a single root Application.

**24. What is Argo Rollouts?**
*Answer:* A Kubernetes controller and set of CRDs (like `Rollout`) that provide advanced deployment capabilities such as Blue/Green, Canary, and progressive delivery, which native Kubernetes Deployments lack. It integrates seamlessly with ArgoCD.

**25. What is Flux (FluxCD)?**
*Answer:* A CNCF graduated GitOps toolset for Kubernetes. It focuses heavily on a modular, component-based architecture (using the GitOps Toolkit) and native Kubernetes API integration without a heavy UI.

**26. How do Flux and ArgoCD differ primarily?**
*Answer:* ArgoCD is known for its monolithic architecture and robust UI dashboard, making it very user-friendly. Flux is designed as a set of highly composable, lightweight microservices (source-controller, kustomize-controller) and is heavily CLI/CRD driven.

**27. What is Crossplane?**
*Answer:* A Kubernetes add-on that extends the K8s API to manage cloud infrastructure (AWS RDS, S3, IAM) via declarative manifests.

**28. How does Crossplane enable GitOps for Cloud Infrastructure?**
*Answer:* By representing cloud resources as Kubernetes CRDs, tools like ArgoCD or Flux can manage AWS infrastructure the exact same way they manage Kubernetes Deployments, unifying app and infrastructure delivery under one GitOps workflow.

**29. What is the `ApplicationSet` controller in ArgoCD?**
*Answer:* It automates the generation of ArgoCD `Applications` based on templates and generators (like a Git directory generator or Cluster generator). It's used for deploying the same app across hundreds of clusters simultaneously.

**30. What is a GitOps webhook?**
*Answer:* While GitOps agents typically poll Git for changes (e.g., every 3 minutes), configuring a Git webhook allows the Git provider (GitHub/GitLab) to actively notify the agent of a commit, triggering near-instantaneous reconciliation.

## Security, Compliance & Secrets Management (31-40)

**31. Why shouldn't you commit Kubernetes Secrets directly to a Git repository?**
*Answer:* Kubernetes Secrets are only base64 encoded, not encrypted. Committing them to Git exposes sensitive credentials, API keys, and passwords to anyone with read access to the repository.

**32. What is Sealed Secrets (Bitnami)?**
*Answer:* A tool that uses asymmetric encryption to encrypt a Kubernetes Secret into a `SealedSecret` CRD. This encrypted file *can* be safely committed to Git. A controller inside the cluster (which holds the private key) decrypts it back into a standard Secret.

**33. What is SOPS (Secrets OPerationS)?**
*Answer:* An open-source tool by Mozilla that encrypts values in YAML/JSON files using AWS KMS, GCP KMS, or PGP. It integrates with Flux and ArgoCD to decrypt the files on-the-fly during reconciliation.

**34. How does the External Secrets Operator work in a GitOps flow?**
*Answer:* Instead of storing encrypted secrets in Git, you store an `ExternalSecret` CRD in Git. This CRD tells the operator to fetch the actual secret value from an external vault (like AWS Secrets Manager or HashiCorp Vault) and dynamically create the K8s Secret inside the cluster.

**35. How does GitOps improve auditability and compliance (like SOC2)?**
*Answer:* Every change to the infrastructure is a Git commit. Git provides a cryptographic, immutable ledger of who made the change, when it was made, what the exact diff was, and who approved the Pull Request.

**36. How do you implement least privilege in a GitOps workflow?**
*Answer:* Developers are given access to the Git repository, but their direct `kubectl` access to the production cluster is revoked. The GitOps agent handles the deployments, acting as the sole authorized deployer.

**37. How do you prevent a malicious developer from deploying a privileged container via GitOps?**
*Answer:* By combining GitOps with Policy-as-Code admission controllers like OPA Gatekeeper or Kyverno. Even if ArgoCD attempts to sync a malicious manifest from Git, the cluster's admission controller will reject it.

**38. What is branch protection in the context of GitOps?**
*Answer:* Enforcing rules on the infrastructure Git repository (e.g., requiring at least one code review, passing CI checks, and blocking direct pushes to the `main` branch) to ensure only peer-reviewed, valid configurations reach the cluster.

**39. How do you securely provide ArgoCD/Flux access to a private Git repository?**
*Answer:* By providing the agent with an SSH deploy key or a short-lived GitHub App token/Personal Access Token stored securely as a Kubernetes Secret in the agent's namespace.

**40. What is Drift Detection alerting, and why is it a security feature?**
*Answer:* If an attacker gains access to the cluster and manually creates a backdoor pod, ArgoCD detects this as "Out of Sync" (drift) because it's not in Git. Alerting on this provides immediate intrusion detection for unauthorized infrastructure modifications.

## Advanced Operations & Multi-Cluster (41-50)

**41. How does GitOps facilitate Disaster Recovery?**
*Answer:* Because the entire cluster state (apps, ingress, config) is in Git, recovering a lost cluster is as simple as spinning up a fresh Kubernetes cluster, installing the GitOps agent, and pointing it at the repository. The agent restores the cluster automatically.

**42. How do you manage secrets for multiple environments (Dev/Prod) in GitOps?**
*Answer:* Use environment-specific KMS keys with SOPS, or use External Secrets Operator with different AWS IAM roles (IRSA) for the Dev and Prod clusters so they can only fetch their respective secrets from AWS Secrets Manager.

**43. What is Progressive Delivery, and how does it relate to GitOps?**
*Answer:* It's the process of releasing updates in a controlled, gradual manner (Canary, Blue/Green) using metrics to gauge success. Tools like Flagger or Argo Rollouts monitor Prometheus metrics and automatically update the Git state or traffic weights based on the results.

**44. How do you handle configuration that must differ per cluster (e.g., cluster-name)?**
*Answer:* Use Kustomize overlays for each cluster, or use Helm charts where the `values.yaml` for each cluster passes the specific cluster-name variable into the templates.

**45. What is the challenge with managing Helm chart versions in GitOps?**
*Answer:* If you point ArgoCD to a floating Helm tag (like `^1.2.0`), you lose the strict determinism of GitOps because the chart can change without a Git commit. Best practice is to pin exact versions (e.g., `1.2.4`) in the declarative manifests.

**46. How do you update container image tags automatically in the Git repository?**
*Answer:* Use tools like ArgoCD Image Updater or Flux Image Update Automation. They monitor the container registry for new images matching a policy and automatically create a commit in the Git repository to update the deployment YAML.

**47. Describe a multi-cluster GitOps architecture.**
*Answer:* A centralized management cluster runs ArgoCD. ArgoCD is configured with credentials (kubeconfigs) to multiple downstream worker clusters. The centralized ArgoCD monitors Git and pushes the required states to the respective worker clusters.

**48. What is a "Sync Window" in ArgoCD?**
*Answer:* A feature that allows you to specify time windows when syncs are allowed or denied (e.g., "deny all syncs to production during the weekend" or "only allow syncs between 2 AM and 4 AM").

**49. How do you handle database schema migrations in a GitOps workflow?**
*Answer:* Migrations should be decoupled from app deployment. They are typically executed as Kubernetes `Jobs` via Helm hooks (e.g., `pre-install`, `pre-upgrade`) or InitContainers, ensuring the schema is ready before the new application pods start receiving traffic.

**50. What is a common anti-pattern in GitOps adoption?**
*Answer:* Giving developers broad `kubectl` write access while simultaneously using a GitOps tool. This leads to constant conflict (drift) between manual changes and the automated reconciliation loop, defeating the purpose of a single source of truth.

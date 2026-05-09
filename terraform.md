# 50 Terraform Interview Questions & Answers

## Core Concepts & State Management (1-10)

**1. What is Terraform, and what is its primary use case?**
*Answer:* Terraform is an open-source Infrastructure as Code (IaC) software tool created by HashiCorp. It allows users to define and provision data center infrastructure using a declarative configuration language (HCL).

**2. Explain the concept of "State" in Terraform.**
*Answer:* State is how Terraform maps real-world resources to your configuration, keeps track of metadata, and improves performance for large infrastructures. By default, it's stored in a local file named `terraform.tfstate`.

**3. Why is remote state important?**
*Answer:* Remote state stores the `terraform.tfstate` file in a remote data store (like AWS S3 or HashiCorp Consul). This enables team collaboration, ensures the state file isn't lost if a local machine dies, and keeps sensitive data (stored in plain text within the state) secure.

**4. How do you prevent concurrent executions from corrupting the Terraform state?**
*Answer:* By configuring state locking. If using an AWS S3 backend, you configure a DynamoDB table. Terraform acquires a lock in the table before running `apply` or `plan` and releases it afterward, preventing race conditions.

**5. What happens if a resource is created manually in the AWS console, but it is also defined in Terraform?**
*Answer:* This causes "drift." Terraform is unaware of the manual change. On the next `terraform plan`, Terraform will detect the discrepancy and propose modifying or recreating the resource to match the declarative configuration in the code.

**6. How do you bring manually created infrastructure under Terraform's control?**
*Answer:* Write the resource configuration block in your `.tf` file, then run `terraform import <resource_type>.<resource_name> <provider_resource_id>`. This maps the real-world object to the Terraform state.

**7. What is `terraform refresh`, and is it still necessary?**
*Answer:* `terraform refresh` reads the current settings from all managed remote objects and updates the state to match. In modern Terraform (v0.15+), `refresh` is automatically integrated into the `plan` and `apply` commands, so running it standalone is rarely necessary and actually deprecated.

**8. Explain the difference between `count` and `for_each`.**
*Answer:* Both create multiple instances of a resource. `count` iterates based on an integer (index 0, 1, 2). If you remove an item from the middle of a list, Terraform shifts the indices, potentially destroying and recreating unrelated resources. `for_each` iterates over a map or set of strings, creating resources keyed by strings, which is much safer for additions/removals.

**9. What are Terraform Providers?**
*Answer:* Providers are plugins that Terraform uses to interact with cloud providers, SaaS providers, and other APIs (e.g., AWS, Azure, GitHub, Kubernetes). They understand API interactions and expose resources.

**10. What is a Provider Alias?**
*Answer:* It allows you to define multiple configurations for the same provider within a single module. Useful for deploying resources to multiple AWS regions or multiple AWS accounts within the same Terraform run.

## Modules, Workspaces & HCL (11-20)

**11. What is a Terraform Module?**
*Answer:* A module is a container for multiple resources that are used together. Every Terraform configuration has at least one module (the root module). You can create child modules to encapsulate and reuse infrastructure logic.

**12. How do you pass data into and out of a module?**
*Answer:* You pass data into a module using input `variable` blocks. You pass data out of a module using `output` blocks.

**13. What is the difference between a `variable` and a `local`?**
*Answer:* A `variable` is an input parameter that can be overridden by the user at runtime (via CLI, `.tfvars` file, or environment variables). A `local` is an internal constant or computed expression scoped to the module that cannot be overridden from the outside.

**14. Explain Terraform Workspaces.**
*Answer:* Workspaces allow you to manage multiple states associated with a single configuration directory. When you run `terraform workspace new dev`, Terraform creates a separate state file (e.g., `env:/dev/`) preventing the `dev` state from overlapping with `prod`.

**15. When should you use Workspaces vs. Separate Directories for environments?**
*Answer:* HashiCorp recommends Workspaces for environments that are structurally identical but differ in scale/variables (like parallel test environments). For rigid separation between Dev, Staging, and Prod where blast radius containment and different backend credentials are required, separate directories are preferred.

**16. What is a `data` source in Terraform?**
*Answer:* A `data` source allows Terraform to fetch read-only information defined outside of the current Terraform configuration (e.g., looking up the latest AWS AMI ID, or fetching a VPC ID created by another team).

**17. What does the `depends_on` meta-argument do?**
*Answer:* Terraform automatically creates a dependency graph based on variable references. `depends_on` is used to explicitly specify a hidden dependency that Terraform cannot infer, forcing Resource B to wait for Resource A to complete creation.

**18. What is the `lifecycle` block used for?**
*Answer:* It controls the behavior of resource creation and destruction. Common arguments include `create_before_destroy` (creates the new resource before deleting the old one for zero downtime), `prevent_destroy` (protects against accidental deletion), and `ignore_changes` (tells Terraform to ignore drift on specific arguments).

**19. How do you implement conditional logic in Terraform?**
*Answer:* Using the ternary operator: `condition ? true_val : false_val`. For example, conditionally creating a resource: `count = var.create_resource ? 1 : 0`.

**20. What is dynamic block configuration in Terraform?**
*Answer:* The `dynamic` block allows you to dynamically construct repeatable nested blocks (like `ingress` rules within an AWS Security Group) based on a complex variable, such as a list of objects, rather than hardcoding each block.

## Advanced State Operations & Refactoring (21-30)

**21. You need to rename a resource in your `.tf` code. How do you avoid destroying and recreating it?**
*Answer:* Use the `moved` block (introduced in Terraform 1.1) to tell Terraform that the resource's logical name has changed. Alternatively, use the CLI command: `terraform state mv aws_instance.old_name aws_instance.new_name`.

**22. What is `terraform taint`?**
*Answer:* It marks a resource as degraded or damaged, forcing Terraform to destroy and recreate it on the next `apply`. (Note: `terraform taint` is deprecated in v0.15.2+; use `terraform apply -replace="resource_address"` instead).

**23. A team member accidentally deleted the remote state file from S3. What do you do?**
*Answer:* If S3 versioning was enabled (which is a strict best practice for remote backends), restore the previous version of the `terraform.tfstate` file from S3. If versioning was off, you must painstakingly use `terraform import` to rebuild the state mapping for every resource.

**24. How do you safely remove a resource from Terraform's management without destroying the actual cloud resource?**
*Answer:* Use the `terraform state rm <resource_address>` command. Terraform will forget about the resource, and the next `apply` will not affect the real-world object.

**25. You are managing thousands of resources in one state file and `terraform plan` is too slow. How do you solve this?**
*Answer:* Refactor the monolithic state into smaller, decoupled state files (e.g., split by networking, database, application layer). Use `terraform_remote_state` data sources to pass outputs between the decoupled configurations.

**26. What is a `terraform_remote_state` data source?**
*Answer:* It allows one Terraform configuration to read the root-level `output` values from another Terraform configuration's state file (e.g., the Application code reading the VPC ID outputted by the Networking code).

**27. Explain the `target` flag in `terraform apply`.**
*Answer:* `terraform apply -target=resource_address` instructs Terraform to only plan and apply changes to that specific resource and its dependencies, ignoring the rest of the configuration. Useful for recovering from partial failures or applying urgent hotfixes.

**28. How does Terraform handle secrets?**
*Answer:* Badly. By default, Terraform stores variables and resource attributes in plain text in the state file. You must secure the state file (S3 encryption, strict IAM access). For providing secrets, use environment variables (`TF_VAR_password`) or data sources pointing to AWS Secrets Manager/Vault, avoiding hardcoded secrets in `.tf` files.

**29. What is Terraform Cloud (TFC)?**
*Answer:* A managed service by HashiCorp that provides remote state storage, secure variable management, centralized run execution (agents), RBAC, and a private module registry for enterprises.

**30. How do you share local variables between multiple `.tf` files in the same directory?**
*Answer:* All `.tf` files in a directory are concatenated and evaluated as a single module. A `local` defined in `variables.tf` can be freely referenced in `main.tf` or `outputs.tf` without any special import syntax.

## Provisioners, Functions & The CLI (31-40)

**31. What are Terraform Provisioners?**
*Answer:* Scripts or commands executed on a local or remote machine during resource creation or destruction (e.g., `local-exec`, `remote-exec`). 

**32. Why does HashiCorp say Provisioners are a "Last Resort"?**
*Answer:* Because provisioner actions are not tracked by Terraform state. If a provisioner fails, Terraform marks the resource as "tainted." Declarative configuration management tools (Ansible, Chef) or cloud-native user-data (cloud-init, Packer AMIs) are far more reliable.

**33. What is the `null_resource`?**
*Answer:* A resource that implements the standard resource lifecycle but takes no further action. It is often used to trigger provisioners or execute local scripts when a specific `trigger` condition changes.

**34. What has largely replaced `null_resource` in modern Terraform?**
*Answer:* The `terraform_data` resource (introduced in v1.4) natively replaces `null_resource` without requiring the initialization of the `null` provider.

**35. Explain the `coalesce` function.**
*Answer:* `coalesce(val1, val2, ...)` takes any number of arguments and returns the first one that isn't null or an empty string. Useful for setting default fallbacks.

**36. Explain the `merge` function.**
*Answer:* `merge(map1, map2)` takes two or more maps and combines them into a single map. Useful for dynamically combining default tags with environment-specific tags.

**37. How do you format Terraform code to a canonical standard?**
*Answer:* Run the `terraform fmt` command. (Tip: `terraform fmt -recursive` formats all files in subdirectories).

**38. What does `terraform validate` do?**
*Answer:* It checks whether the configuration is syntactically valid and internally consistent (e.g., checking for missing variables or incorrect attribute references), without accessing any remote state or cloud provider APIs.

**39. How do you view a graph of your Terraform dependencies?**
*Answer:* `terraform graph` outputs the visual dependency graph of Terraform resources in DOT format, which can be rendered into an image using tools like Graphviz.

**40. What is the Terraform Registry?**
*Answer:* A centralized, public repository (registry.terraform.io) maintained by HashiCorp where users can find and distribute Providers (AWS, Azure) and Modules (VPC, EKS).

## Security, Testing, & Enterprise Workflows (41-50)

**41. What is Terratest?**
*Answer:* An open-source Go library developed by Gruntwork that makes it easier to write automated tests for Terraform. It actually deploys the infrastructure, verifies it works using APIs/SSH, and then destroys it (`defer terraform.Destroy`).

**42. How does `tfsec` or `checkov` fit into a Terraform pipeline?**
*Answer:* They are static analysis security testing (SAST) tools. They scan your `.tf` code *before* deployment (e.g., during a Pull Request) to find security misconfigurations (like public S3 buckets or unencrypted EBS volumes).

**43. What is OPA / Rego in the context of Terraform?**
*Answer:* Open Policy Agent (OPA) uses the Rego language to define Policy-as-Code. You can convert a `terraform plan` output to JSON and evaluate it against OPA to block deployments that violate organizational rules (e.g., "Deny creation of EC2 instances larger than t3.large").

**44. What is HashiCorp Sentinel?**
*Answer:* Sentinel is HashiCorp's proprietary Policy-as-Code framework built directly into Terraform Cloud and Terraform Enterprise, serving a similar function to OPA.

**45. Explain how a GitOps/CI-CD workflow looks for Terraform.**
*Answer:* Developer opens a PR -> CI runs `terraform fmt`, `tflint`, `tfsec`, and `terraform plan`. The PR is reviewed alongside the Plan output. -> PR is merged to `main` -> CD pipeline runs `terraform apply -auto-approve` to deploy the infrastructure.

**46. What is Atlantis?**
*Answer:* An open-source tool for Terraform pull request automation. It listens for webhooks from GitHub/GitLab, runs `terraform plan`, and posts the output directly as a comment on the PR. Users comment `atlantis apply` to deploy.

**47. What is `tflint`?**
*Answer:* A Terraform linter that checks for possible errors, deprecated syntax, and provider-specific best practices (e.g., warning if you hardcode an AWS instance type that doesn't exist).

**48. Why should you pin provider versions?**
*Answer:* To ensure deterministic deployments. If a provider introduces a breaking API change in a major version bump, your `terraform apply` might fail or behave unexpectedly unless you have constrained the version (e.g., `version = "~> 4.0"`).

**49. What is a "partial backend configuration"?**
*Answer:* Instead of hardcoding the S3 bucket name and DynamoDB table in the `.tf` file for remote state, you define an empty `backend "s3" {}` block and pass the backend configuration dynamically at runtime via `terraform init -backend-config="bucket=my-bucket"`. This is crucial for reusing code across multiple environments.

**50. What is Terragrunt?**
*Answer:* A thin wrapper for Terraform developed by Gruntwork. It provides tools to keep configurations DRY (Don't Repeat Yourself), manage remote state dynamically across multiple AWS accounts, and handle multi-module dependencies easily.

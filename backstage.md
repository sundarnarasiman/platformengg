# Backstage Interview Q&A for Platform Engineers

This document contains 50 interview questions and answers focused on Backstage, the open-source platform for building developer portals, categorized by core features and architecture.

## Core Concepts & Architecture (1-10)

**1. What is Backstage, and what primary problem does it solve?**
*Answer:* Backstage is an open platform for building developer portals, originally created by Spotify. It solves the problem of infrastructure fragmentation and cognitive overload by centralizing services, software templates, documentation, and tooling into a single, cohesive developer experience (IDP - Internal Developer Portal).

**2. Describe the high-level architecture of a Backstage application.**
*Answer:* Backstage is a monorepo consisting of three main parts: 
1) A Frontend app (React/TypeScript) built on a plugin architecture.
2) A Backend (Node.js/Express) that serves APIs, connects to databases (Postgres/SQLite), and powers backend plugins.
3) A Database that stores state for the catalog, scaffolder, and other plugins.

**3. What is the role of the `@backstage/core-plugin-api`?**
*Answer:* It provides the core APIs and utilities for building Backstage frontend plugins. It includes hooks for routing, configuration access, error handling, and interacting with the Backstage identity system.

**4. How does Backstage handle authentication and authorization?**
*Answer:* Authentication in Backstage is handled by identity providers (like GitHub, Google, Okta, Auth0) integrated via backend auth providers. Authorization is managed by the Backstage Permission framework, which evaluates policies to determine if a user or group has access to specific resources or actions within the catalog or plugins.

**5. What is an Internal Developer Portal (IDP), and why is it important for platform engineering?**
*Answer:* An IDP is a self-service hub that abstracts the complexities of infrastructure and tooling. For platform engineering, it improves developer velocity, enforces organizational best practices ("golden paths"), and reduces the time needed to scaffold, deploy, and monitor new microservices.

**6. Explain the concept of "Golden Paths" in the context of Backstage.**
*Answer:* A Golden Path is an opinionated, well-supported, and automated route to building and deploying software. Backstage facilitates this via Software Templates (Scaffolder), ensuring new services are created with CI/CD, monitoring, and security tooling pre-configured according to organizational standards.

**7. How is configuration managed in a Backstage application?**
*Answer:* Backstage uses `app-config.yaml` files. It supports merging configurations from multiple files (e.g., `app-config.production.yaml`) and reading environment variables at runtime, allowing seamless deployments across dev, staging, and prod environments.

**8. What database does Backstage recommend for production use?**
*Answer:* PostgreSQL. While SQLite is supported for local development and testing, PostgreSQL provides the required concurrency, reliability, and features needed for a production deployment.

**9. How do Frontend and Backend plugins communicate in Backstage?**
*Answer:* The Frontend plugin makes HTTP/REST requests to the Backend plugin. Backstage provides an `ApiRef` system and `DiscoveryApi` in the frontend to correctly resolve the backend plugin's URL, handling authentication tokens automatically if configured.

**10. How would you deploy Backstage to Kubernetes?**
*Answer:* You would containerize the Node.js application using Docker, define Kubernetes Deployments and Services, configure an Ingress controller for routing, deploy a managed PostgreSQL database (like AWS RDS), and use external storage (like S3) for TechDocs. Helm charts are commonly used for this.

## The Software Catalog (11-20)

**11. What is the Backstage Software Catalog?**
*Answer:* The Catalog is a centralized system that keeps track of ownership and metadata for all the software in your ecosystem (services, websites, libraries, data pipelines). It is built around the concept of metadata YAML files living alongside the code.

**12. Explain the core Entity kinds in the Backstage Catalog model.**
*Answer:* 
*   `Component`: A piece of software (Service, Website, Library).
*   `API`: An interface boundaries between components.
*   `Resource`: Infrastructure needed to operate a component (Database, S3 bucket).
*   `System`: A collection of entities cooperating to perform a function.
*   `Domain`: A collection of systems that share terminology or purpose.
*   `User` & `Group`: Represents individuals and teams owning the software.

**13. Where is the source of truth for the Software Catalog?**
*Answer:* The source of truth is the code repository itself. Backstage reads `catalog-info.yaml` files located in the root of each Git repository, allowing developers to manage catalog metadata using standard GitOps workflows.

**14. What is a Location in the context of the Catalog?**
*Answer:* A `Location` is an entity kind that tells Backstage where to find other catalog entities. It's essentially a pointer to a URL, Git repository, or file path containing `catalog-info.yaml` files.

**15. How does Backstage keep the Catalog in sync with changes in Git repositories?**
*Answer:* The Backstage catalog backend uses processors and providers. Providers (like GitHub, GitLab, Bitbucket integration) actively discover and ingest repositories matching a pattern, or use webhooks to immediately update the catalog when a `catalog-info.yaml` file changes.

**16. What is an Entity Reference in Backstage?**
*Answer:* It's a string used to identify an entity in the catalog, formatted as `[<kind>:][<namespace>/]<name>` (e.g., `component:default/my-service`).

**17. How do you define ownership of a component in `catalog-info.yaml`?**
*Answer:* Ownership is defined in the `spec.owner` field, which references a `User` or `Group` entity (e.g., `owner: group:payment-team`). This is crucial for tracing operational responsibility.

**18. What are Catalog Processors?**
*Answer:* Processors are customizable backend functions that run during the entity ingestion loop. They can read raw data, validate it, mutate entities, or generate new entities based on the input (e.g., automatically generating API entities from a single Component).

**19. How do you handle orphaned entities in the Catalog?**
*Answer:* Orphaned entities occur when the source `catalog-info.yaml` is deleted from Git but the entity remains in the database. Backstage marks them as orphaned, and users can manually clean them up via the UI, or you can write custom background tasks to prune them.

**20. Can the Catalog ingest data from external systems besides Git?**
*Answer:* Yes. Using custom Entity Providers, you can ingest data from external sources like LDAP/Active Directory for user/group data, AWS for infrastructure resources, or ServiceNow for existing CMDB data.

## Software Templates & Scaffolder (21-30)

**21. What is the Backstage Scaffolder (Software Templates)?**
*Answer:* The Scaffolder is a core feature that allows platform engineers to define templates for creating new software projects. It automates repository creation, boilerplate code generation, and CI/CD setup, ensuring new projects follow organizational best practices.

**22. Describe the structure of a Backstage Software Template.**
*Answer:* A template is a YAML file of kind `Template`. It has a `spec` that defines:
1) `parameters`: JSON Schema defining the form the user fills out in the UI.
2) `steps`: An array of actions to execute sequentially (e.g., fetch template, rename files, publish to github).
3) `output`: Links to the generated resources (repo, catalog entity) displayed to the user upon completion.

**23. What templating language does the Scaffolder use for rendering files?**
*Answer:* Backstage uses Nunjucks (similar to Jinja2) to inject user parameters into the skeleton files and directory names during the scaffolding process.

**24. What is a Scaffolder Action?**
*Answer:* An action is a reusable, atomic backend function executed as a step in a template. Built-in actions include `fetch:template` (download boilerplate), `publish:github` (create a repo), and `catalog:register` (register the new entity).

**25. How do you create a Custom Scaffolder Action?**
*Answer:* You write a TypeScript function using the `createTemplateAction` API in the backend. You define the action's schema (inputs/outputs) and a `handler` function that contains the custom logic (e.g., making an API call to create an AWS S3 bucket).

**26. How do you secure Scaffolder templates?**
*Answer:* You can use the Backstage Permission framework to restrict which users or groups can view or execute specific templates based on their tags or ownership.

**27. What is `fetch:cookiecutter` and when would you use it?**
*Answer:* It's a scaffolder action that uses the popular Python tool Cookiecutter to render templates. You would use it if your organization already has a large existing library of Cookiecutter templates that you want to integrate into Backstage without rewriting them.

**28. How does the Scaffolder handle authentication to create repositories on GitHub/GitLab?**
*Answer:* The Backstage backend uses integrations configured in `app-config.yaml` with a personal access token (PAT) or a GitHub App to authenticate and perform actions like creating repositories on behalf of the organization.

**29. Can a template execute a local script on the Backstage backend server?**
*Answer:* While technically possible by writing a custom action that uses Node's `child_process`, it is highly discouraged due to security risks, scalability issues, and container immutability principles. Actions should rely on APIs.

**30. How do you debug a failing Scaffolder template?**
*Answer:* Backstage provides a real-time log viewer in the UI during execution. If an action fails, the logs will show the exact step and error message. You can also review the backend Node.js logs.

## TechDocs (31-40)

**31. What is Backstage TechDocs?**
*Answer:* TechDocs is a "docs-like-code" solution built into Backstage. It allows developers to write documentation in Markdown alongside their code, which Backstage generates into static HTML and serves within the developer portal.

**32. What engine powers the generation of TechDocs?**
*Answer:* MkDocs. Backstage runs MkDocs under the hood to compile Markdown files into HTML, often utilizing the highly customizable `mkdocs-material` theme.

**33. How does a developer enable TechDocs for their repository?**
*Answer:* They must add an `mkdocs.yml` file and a `docs/` folder containing Markdown files to their repository. Then, they add the `backstage.io/techdocs-ref: dir:.` annotation to their `catalog-info.yaml` file.

**34. Explain the difference between "local" and "recommended" TechDocs generation.**
*Answer:* "Local" generation means the Backstage backend server runs MkDocs to generate the HTML when a user requests the docs. "Recommended" generation offloads this to a CI/CD pipeline (e.g., GitHub Actions), which generates the HTML and publishes it to a cloud storage bucket (like AWS S3 or GCS), where Backstage reads it.

**35. Why is CI/CD generation recommended for production TechDocs?**
*Answer:* Relying on the Backstage backend to generate docs on-the-fly does not scale well. It consumes significant CPU/memory, introduces latency, and requires Python/MkDocs dependencies to be installed on the Backstage server container.

**36. How do you secure access to TechDocs stored in AWS S3?**
*Answer:* Ensure the S3 bucket is private. Configure Backstage with an IAM Role (or IAM User credentials) that has `s3:GetObject` permissions. Backstage will securely fetch the HTML and proxy it to the authenticated user via the UI.

**37. How can you include diagrams in TechDocs?**
*Answer:* TechDocs supports Mermaid.js natively. Furthermore, because MkDocs powers it, you can install plugins like PlantUML in your MkDocs environment to render complex architectural diagrams from code.

**38. What is the Backstage Search feature, and how does it integrate with TechDocs?**
*Answer:* Backstage Search is an extensible framework for indexing catalog entities and documentation. The TechDocs search collator indexes all generated HTML content, allowing users to perform full-text searches across all technical documentation in the organization globally.

**39. Can TechDocs render OpenAPI or Swagger specifications?**
*Answer:* While TechDocs (MkDocs) is primarily for Markdown, Backstage has a separate `api-docs` plugin specifically designed to render OpenAPI, GraphQL, and AsyncAPI specifications registered as `API` entities in the catalog.

**40. How do you manage global documentation (docs not tied to a specific microservice)?**
*Answer:* Create a dedicated Git repository for the global documentation, define it as a `Component` or `System` entity in a `catalog-info.yaml`, and treat it exactly like service documentation.

## Plugins & Ecosystem (41-50)

**41. What makes Backstage's UI highly customizable?**
*Answer:* The UI is composed entirely of React components provided by plugins. Platform engineers can compose the App layout, Entity pages, and sidebars by importing and placing these React components wherever they choose in the application code.

**42. How do you share state between different Backstage plugins?**
*Answer:* While plugins are meant to be isolated, they can share state using standard React Contexts, or by utilizing Utility APIs provided by the `@backstage/core-plugin-api` framework that are injected throughout the app.

**43. Give three examples of popular open-source Backstage plugins.**
*Answer:* 
1) **Kubernetes:** Surfaces cluster status, pods, and errors directly on the service page.
2) **CircleCI/GitHub Actions:** Shows CI/CD build statuses.
3) **PagerDuty:** Shows who is on-call and allows acknowledging incidents.

**44. What is the difference between an App and a Plugin in Backstage architecture?**
*Answer:* A Plugin is an isolated piece of functionality (frontend component and/or backend router). The App is the shell that wires all the chosen plugins together, configures them, and provides the routing and overall theme.

**45. If a plugin requires a database to store its own state, how is this handled?**
*Answer:* The Backstage backend provides a Database API (`PluginDatabaseManager`). When developing a backend plugin, you request a database connection from this manager, which automatically provisions a logical database or schema for that specific plugin within the main PostgreSQL instance.

**46. How do you implement a custom theme in Backstage?**
*Answer:* You create a custom theme object overriding Material-UI (MUI) variables (colors, typography) using `createTheme` from Backstage. You then pass this theme into the `AppProvider` in the frontend initialization file (`App.tsx`).

**47. What is the purpose of "Annotations" in Backstage?**
*Answer:* Annotations are key-value pairs added to an entity's `catalog-info.yaml`. They are heavily used by plugins to link an entity to external systems (e.g., `github.com/project-slug` links the entity to a GitHub repo to fetch PRs or issues).

**48. How do you handle frontend plugin testing?**
*Answer:* Use standard React testing tools like Jest and React Testing Library. Backstage provides utilities like `renderInTestApp` to wrap components in necessary contexts (routing, API registry) for isolated testing.

**49. What is Backstage "Soundcheck" (or similar Tech Radar / Scorecard plugins)?**
*Answer:* These plugins evaluate software against organizational standards (e.g., "Does it have a PagerDuty integration?", "Is test coverage > 80%?"). It provides a gamified, visual scorecard to drive engineering health and alignment.

**50. What is the long-term maintenance challenge with Backstage, and how do you mitigate it?**
*Answer:* Because you are building a custom Node/React app pulling in hundreds of Backstage packages, keeping it upgraded can be challenging due to breaking changes. Mitigation involves regular, incremental upgrades using the `@backstage/cli versions:bump` tool, reading release notes carefully, and having robust automated tests for custom plugins.

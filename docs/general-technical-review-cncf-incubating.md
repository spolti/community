# General Technical Review - KServe / Incubating

- **Project:** KServe
- **Project Version:** v0.15.0
- **Website:** https://kserve.github.io/website/
- **Date Updated:** 2025-05-07
- **Template Version:** v1.0
- **Description:** Highly scalable and standards based Model Inference Platform on Kubernetes
- **Reviewers:** Dan Sun, Yuan Tang, Sivanantham Chinnaiyan, Johnu George


## Day 0 - Planning Phase

### Scope

  * Describe the roadmap process, how scope is determined for mid to long term features, as well as how the roadmap maps back to current contributions and maintainer ladder?
  
    - **Vision and Strategic alignment:**
      - The roadmap start with the project's vision and strategic goals for scalable inference with predictive and generative inference on k8s. 
      - These are often defined by leads and core maintainers or in alignment with the feedback from the community.
    - **Gathering Feedbacks:**
      - inputs are collected from various stakeholders including github issues, contributor proposals
      - Usage trends and telemetry
      - Ecosystem changes
    - **Defining the Scope:**
      - Mid-term: prioritize by immediate impact and feasibility
      - Long-term: Focus on larger initiative or architecture changes
    - **Map to Contributions:**
      - Issues are tagged with milestone labels and added to the release project boards
      - Working group to align with the roadmap initiatives
    - **Map to Contributor Ladder:** 
      - Maintainer signal: Contributor involved in roadmap aligned work demonstrate sustained impacts
      - Leader role: mid or long term roadmap items often need lead contributor who can drive the initiatives

  * Describe the target persona or user(s) for the project?
    **ML Platform Engineer/MLOps Engineer**
    - Build scalable and reliable infrastructure for model deployment and inference.
    - Enable data scientists to deploy models without managing infrastructure.
    - Ensure observability, security and autoscaling for inference services.
    - Ensure high availability, scalability.

    **Data Scientist/ML Researcher**
    - Deploy models easily to get predictions.
    - Run experiments and iterate quickly on model versions. 
    - Access model metrics or explanations for outputs.

  * Explain the primary use case for the project. What additional use cases are supported by the project?
    - A declarative API to define Inference Services or graph for deploying models or pipeline.
    - Multi-framework support out-of-the-box Tensorflow, Pytorch, XGBoost, ONNX, Hugging Face(LLMs), Scikit-Learn, LightGBM.
    - Autoscaling, including scale-to-zero and GPU support for resource utilization efficiency.
    - Seamless integration with K8s native tools like Istio, Knative, Prometheus.

    Additional features supported:
    - Canary rollout
    - Model explainability
    - Inference logging and tracing
    - Custom predictor, pre/post processing
    - Multi-model serving, ModelMesh

  * Explain which use cases have been identified as unsupported by the project.  
    - KServe primarily focus on Inference, not training or fine-tunings.
    - KServe is designed to be part of the larger MLOps ecosystem, model pipeline orchestration is expected to be handled by Kubeflow Pipelines or other projects.
    - KServe is designed for running in k8s environment, non-k8s environment is not supported.

  * Describe the intended types of organizations who would benefit from adopting this project. (i.e. financial services, any software manufacturer, organizations providing platform engineering services)?  
    - Enterprise with ML platform teams running on k8s.
    - Mid-size companies for scaling ML inference infrastructure.
    - Organizations requiring multi-framework and multi-tenant support.
    - High regulated industries where need auditable and explainable ML outputs.
    - Security-conscious industries where need Istio, OPA to secure the model inference.

  * Please describe any completed end user research and link to any reports.
    - KServe has wide range of adopters including cloud providers, end users in finance, real-estate industries.

### Usability

  * How should the target personas interact with your project?  
      * The data scientist persona prefers a straightforward and easy way to deploy models without getting involved in the complexities of infrastructure. They can simply specify the model storage location in the InferenceService configuration, allowing the model to be loaded and deployed without needing to understand the underlying platform. 
      * ML Platform engineers and advanced users have the capability to customize the configuration in greater detail, enabling them to gather more insights about the infrastructure for optimizing workloads and enhancing hardware utilization.  
  * Describe the user experience (UX) and user interface (UI) of the project.  
      * The datascientist prefers to a single click deployment either by uploading a model artifact or pointing to a location where model is stored. 
      * ML Platform engineer looks at different system and model metrics to monitor the infrastructure utilization and performance so as to provide the right SLA for the Inference. 
  * Describe how this project integrates with other projects in a production environment.
      * This project fits into the serving phase of the entire MLOps lifecycle. Datascientist trains the model using popular tools like Kubeflow Training Operator and registers the trained model artifact to a model registry. Then, the trained model is put into production at scale using KServe. Serving phase is monitored over time to detect the model drift which will trigger a finetuning workflow at regular intervals creating more model versions.  

### Design

  * Explain the design principles and best practices the project is following.   
  
    - **Kubernetes Native**
      - Leverages Kubernetes custom resource definitions for model deployment.
      - Integrates natively with k8s pods, services and autoscaler.
      - Enable declarative management and gitops-style deployment for ML models.
    - **Unified Control Plane**
      - A single API for deploying model from different ML frameworks.
    - **Security and Multi-Tenancy**
      - Integrates Istio for securing traffic for model inference.
      - Use Kubernetes namespace for tenant isolation with configured resources limits and RBAC.
    - **Extensibility**
      - Allow users to customize predictor, pre/post processing and explanation conforming to the custom runtime contract.
    - **Cloud and Platform Agnostic**
      - Runs on any cloud provider or on-prem kubernetes cluster.
      - Avoids vendor lock-in while enabling hybrid or multi-cloud MLOps.

  * Outline or link to the project’s architecture requirements? Describe how they differ for Proof of Concept, Development, Test and Production environments, as applicable.  
    KServe serverless mode depends on Knative for autoscaling(scale from zero) and revision based canary rollout. Both serverless and raw deployment mode depends on Ingress Gateway(Istio Ingress Gateway/Envoy Gateway) if user has external traffic getting into the KServe cluster. If service mesh is a requirement for secured setup, Istio is required for mTLS. 
 
    - Proof of Concept: user can run the quick installation script to install KServe in local environment.
    - Development/Test: developer can install KServe on Kind or Minikube for quick development iteration.
    - KServe is primarily focusing on deploying in the production environment on k8s cluster using the officially released KServe helm chart.

  * Define any specific service dependencies the project relies on in the cluster.  
    - Kubernetes APIServer for creating/updating/deleting KServe custom resources.
    - Cert Manager for managing KServe webhook certificates.

  * Describe how the project implements Identity and Access Management.  
    - KServe storage component use Kubernetes secrets to store credentials and support cloud identity and IAM for authentication.
    - KServe data plane are secured by Istio for mTLS and authenticated/authorized using security policies.

  * Describe how the project has addressed sovereignty.  
    - KServe is designed to be cloud and platform agnostic, running on any Kubernetes cluster (on-premises or any cloud provider). It avoids vendor lock-in by using open standards and Kubernetes-native APIs. All data and model artifacts remain within the adopter's control, and integrations with external storage or identity providers are optional and configurable by the user.

  * Describe any compliance requirements addressed by the project.  
    - KServe supports compliance with industry standards by leveraging Kubernetes RBAC, network policies, and integration with service mesh (Istio) for secure, auditable traffic. It enables multi-tenancy, namespace isolation, and supports secure storage of credentials. KServe is used in regulated industries and can be configured to meet requirements such as data residency, access control, and auditability, but specific compliance certifications are the responsibility of the adopter's deployment.

  * Describe the project’s High Availability requirements.  
    
    - Control Plane HA: KServe controller can be deployed with multiple replicas across different nodes and availability zones. It uses leader election to ensure only one instance is active at a time, while others are on standby. This ensures that if one instance fails, another can take over without downtime.
    - Data Plane HA: KServe supports multiple replicas of model serving pods, allowing for load balancing and failover. It can be configured with Kubernetes HPA, KEDA or Knative autoscaler to scale based on traffic. For production, it is recommended to use a highly available storage backend for models (e.g., S3, GCS) to ensure data availability.
    - KServe leverages Kubernetes' built-in mechanisms for pod replication, self-healing, and rolling updates. For production, it is recommended to run at least 2-3 replicas of the controller across different nodes and use highly available storage backends for model storage. Integration with Knative and Ingress network solutions like Istio, Envoy Gateway further supports traffic routing and failover.

  * Describe the project’s resource requirements, including CPU, Network and Memory.  
    - The KServe controller requires modest CPU and memory (typically 0.5-1 CPU and 1-2Gi memory per replica). Model serving pods' requirements depend on the model size and framework, with defaults set to 1 CPU and 2Gi memory per pod, but can be tuned per InferenceService. Network requirements depend on inference request volume and model size. For high-throughput or GPU workloads, resource requests and limits should be adjusted accordingly.

  * Describe the project’s storage requirements, including its use of ephemeral and/or persistent storage.  
    - Ephemeral storage is needed for models downloaded local to the Pod.
    - Persistent storage is used for loading models using PV/PVC.
    - KServe control plane does not require persistent storage, except for optional model caching using PV/PVC.

  * Please outline the project’s API Design:  
    * Describe the project’s API topology and conventions
      *Control Plane*:
        - KServe exposes a Kubernetes-native declarative API using Custom Resource Definitions (CRDs), primarily the `InferenceService` resource, to manage model serving workloads. 
        - The API follows Kubernetes conventions for resource specification, status, and metadata, supporting standard CRUD operations via `kubectl` and the Kubernetes API server.
        - The `InferenceService` CRD allows users to define predictors, transformers, and explainers in a single resource, supporting multi-framework and multi-model serving.
        - KServe supports versioned APIs (e.g., `serving.kserve.io/v1beta1`) and uses OpenAPI schema validation for resource definitions.
      *Data Plane*:
        - Inference requests are served via HTTP/gRPC endpoints, following standard inference protocols (e.g., [OpenAI API](https://platform.openai.com/docs/api-reference/introduction), [V2 Open Inference Protocol](https://kserve.github.io/website/master/modelserving/data_plane/v2_protocol/), [KServe V1 protocol](https://kserve.github.io/website/master/modelserving/data_plane/v1_protocol/)), with RESTful conventions for completion, chat, predict, explain and metadata endpoints.
        - The API is designed for extensibility, allowing custom runtimes and components to be integrated via well-defined interfaces.
      - For more details, see the [KServe API Reference](https://kserve.github.io/website/master/reference/api/).
    * Describe the project defaults
      - Deployment mode defaults to serverless
      - Default InferenceService container resource limit with 1 CPU and 2Gi memory
      - Defaults to use Knative Autoscaler  
    * Outline any additional configurations from default to make reasonable use of the project
      - For production, it is recommended to set resource requests and limits based on the model size and expected traffic.
      - Enable Istio or Envoy Gateway for ingress traffic management and security.
      - Set up monitoring and alerting using Prometheus and Grafana for observability.
      - Enable GPU support by specifying node selectors and resource limits for GPU-enabled nodes if serving large or complex models.
      - Set up autoscaling configurations for each InferenceService to handle variable workloads and ensure high availability.
      - Integrate with external object storage (e.g., S3, GCS, Azure Blob) for model storage by providing appropriate storage URIs and Kubernetes secrets for credentials.
      - Use Istio or another service mesh for secure, authenticated, and observable traffic management, including mTLS and traffic splitting for canary rollouts.
      - Enable inference logging, tracing, and Prometheus metrics for monitoring and observability.
      - Customize pre/post-processing and explainers by specifying transformer and explainer containers in the InferenceService spec.
      - Adjust configmaps (e.g., `inferenceservice-config`) for advanced settings such as default runtime images, storage options, and ingress behavior.
      - For production, deploy multiple controller replicas.
      - See the [KServe documentation](https://kserve.github.io/website/master/admin/) for more configuration examples and best practices.
    * Describe any new or changed API types and calls - including to cloud providers - that will result from this project being enabled and used  
      - KServe introduces the `InferenceService` Custom Resource Definition (CRD) as the primary API type for managing model serving workloads on Kubernetes. This is a new resource type not present in vanilla Kubernetes.
      - Additional CRDs include `ServingRuntime` and `ClusterServingRuntime`, which are used to specify and manage custom model runtimes for serving different frameworks or custom containers.
      - The `InferenceGraph` CRD is available for defining complex model chaining and inference pipelines, enabling advanced use cases such as ensemble models or multi-step inference workflows.
      - The `LocalModelCache` CRD allows users to specify models that should be cached locally on nodes for performance optimization and reduced download times.
      - Enabling KServe does not change existing Kubernetes APIs, but adds these new CRDs and related endpoints for managing inference services and model runtimes.
      - For cloud provider integration, KServe supports referencing external model storage (e.g., S3, GCS, Azure Blob) via storage URIs in the `InferenceService` spec. This requires users to provide credentials as Kubernetes secrets, but does not introduce new API types to the cloud provider itself.
      - KServe exposes HTTP/gRPC endpoints for inference, explain, and metadata operations, following standard inference protocols (V1, V2, OpenAI API, etc.).
    * Describe compatibility of any new or changed APIs with API servers, including the Kubernetes API server  
      - All KServe CRDs (InferenceService, ServingRuntime, ClusterServingRuntime, InferenceGraph, LocalModelCache) are implemented using Kubernetes Custom Resource Definitions and are fully compatible with the Kubernetes API server.
      - These CRDs follow Kubernetes API conventions for resource creation, update, deletion, and status reporting, and can be managed using standard Kubernetes tools (kubectl, client libraries, etc.).
      - KServe CRDs are versioned and validated using OpenAPI schemas, ensuring compatibility with Kubernetes admission controllers and API server validation mechanisms.
      - KServe is tested and supported on upstream Kubernetes distributions and is designed to work with standard Kubernetes RBAC, admission webhooks, and API aggregation layers.
      - No changes are made to existing Kubernetes APIs; KServe only extends the API surface by adding new resource types.
    * Describe versioning of any new or changed APIs, including how breaking changes are handled  
      - KServe CRDs (e.g., InferenceService, ServingRuntime, ClusterServingRuntime, InferenceGraph, LocalModelCache) use Kubernetes API versioning conventions, such as `v1alpha1`, `v1beta1`, and `v1` in their API group (e.g., `serving.kserve.io/v1beta1`).
      - New features and changes are introduced in alpha or beta versions before being promoted to stable (`v1`).
      - Conversion webhooks are provided to support seamless migration between CRD versions when breaking changes are introduced.
      - Deprecated fields and versions are announced in release notes and documentation, and are supported for a deprecation period before removal.
      - Backward compatibility is maintained for stable APIs, and breaking changes are only introduced in new major or beta versions, following Kubernetes best practices.
      - Users are encouraged to migrate to newer versions using provided migration documentation.
  * Describe the project’s release processes, including major, minor and patch releases.  
      - KServe follows a structured release process for major, minor, and patch releases, as documented in the [RELEASE_PROCESS_v2.md](https://github.com/kserve/kserve/blob/master/release/RELEASE_PROCESS_v2.md).
      - Each release includes versioning, release notes, and changelogs to communicate new features, bug fixes, and deprecations.
      - Release candidates are published for community testing before final releases.
      - The process includes automated CI/CD checks, validation, and artifact publishing to container registries and GitHub artifacts.
      - Deprecated features and APIs are maintained for a deprecation period before removal.

### Installation

  * Describe how the project is installed and initialized, e.g. a minimal install with a few lines of code or does it require more complex integration and configuration?  
      * For getting started, KServe can be installed via a simple command `curl -s "https://raw.githubusercontent.com/kserve/kserve/release-0.15/hack/quick_install.sh" | bash`
      * For production uses, please refer to the [administration guide](https://kserve.github.io/website/latest/admin/serverless/serverless). Examples:
          * For example, KServe can be installed via Helm `helm install kserve-crd oci://ghcr.io/kserve/charts/kserve-crd --version v0.15.0; helm install kserve oci://ghcr.io/kserve/charts/kserve --version v0.15.0`
          * or via kubectl `kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.15.0/kserve.yaml; kubectl apply --server-side -f https://github.com/kserve/kserve/releases/download/v0.15.0/kserve-cluster-resources.yaml`
  * How does an adopter test and validate the installation?
      * Each of the installation guide mentioned above also includes validation steps. For example, once the InferenceService is created, an adopter can test (details can be found [here](https://kserve.github.io/website/latest/get_started/first_isvc/)):
          * The status of the InferenceService via `kubectl get inferenceservices <inference-service-name> -n <namespace-name>`
          * Perform inference by sending requests to the InferenceService via `curl -v -H "Content-Type: application/json" http://<inference-service-name>.<namespace-name>.${CUSTOM_DOMAIN}/v1/models/<inference-service-name>:predict -d @./iris-input.json`

### Security

  * Please provide a link to the project’s cloud native [security self assessment](https://tag-security.cncf.io/community/assessments/): [security/self-assessment.md](security/self-assessment.md)
  * Please review the [Cloud Native Security Tenets](https://github.com/cncf/tag-security/blob/main/security-whitepaper/secure-defaults-cloud-native-8.md) from TAG Security.  
    * How are you satisfying the tenets of cloud native security projects?  
      * KServe is built with security as a foundational concern. By leveraging Kubernetes-native constructs such as Custom Resource Definitions (CRDs), Role- Based Access Control (RBAC), and Network Policies, KServe integrates seamlessly into secure Kubernetes environments. It also inherits security from underlying components like Istio (for service mesh), Envoy Gateway and Knative (for serverless workloads), enforcing best practices from the start.
      * Default configurations follow Kubernetes security best practices (e.g., non-root containers, minimal privileges, disable privilege escalation). Documentation (ConfigMap) provides clear guidance, but some advanced security configurations may require Kubernetes expertise. Secure defaults are enabled by default, but users can override them if needed.
      * Insecure options (e.g., running privileged containers) require explicit configuration.
      * Users can migrate workloads to more secure configurations without breaking changes, leveraging Kubernetes’ declarative model.
      * Exception handling from provided defaults is possible via modifying the resource definition or through configurations which are exposed in the configmap.
    * Describe how each of the cloud native principles apply to your project.  
      * KServe uses containers as the fundamental unit of deployment. Each model is served through an individual containerized inference service, allowing users to package models along with their dependencies in a consistent, portable way. Whether it’s a custom PyTorch model or a pre-trained Hugging Face transformer or any other framework, KServe runs it in an isolated container, making deployments repeatable and manageable.
      * KServe is designed to run natively on Kubernetes and uses it for orchestration. It dynamically manages the lifecycle of model-serving pods, including provisioning, scaling, upgrading, and termination. It leverages Kubernetes resources like Deployments, Services, and Custom Resource Definitions (CRDs) to integrate deeply into the orchestration layer.
      * KServe adopts a microservices architecture by separating concerns such as prediction, explanation, pre-processing, and post-processing into distinct containerized components. Each function can be independently implemented and plugged into the KServe inference graph, promoting modularity and reusability across different models and workflows.
      * KServe uses Knative for autoscaling In Serverless, including scale-to-zero, where inactive services scale down completely to save resources. In raw deployment model scaling is performed using Kubernetes HPA or KEDA. It supports horizontal scaling based on traffic and custom metrics, and GPU autoscaling is also available for high-performance use cases. This elasticity ensures efficient resource use and responsiveness to demand.
      * KServe services interact via HTTP/gRPC APIs and follow standard inference protocols. This decouples model serving from model training or storage concerns. Users can mix and match model types, runtimes, or even infrastructure (e.g., GPU vs CPU) without tightly coupling components, supporting flexible and scalable deployments.
    * How do you recommend users alter security defaults in order to "loosen" the security of the project? Please link to any documentation the project has written concerning these use cases.  
      * KServe uses Istio’s mTLS by default to secure service-to-service communication. This is documented in [KServe website](https://kserve.github.io/website/master/admin/serverless/servicemesh/#turn-on-strict-mtls-and-authorization-policy). To loosen this security setting, switch Istio PeerAuthentication to PERMISSIVE mode. This allows both encrypted (mTLS) and unencrypted (plaintext) traffic. For example,
       ```yaml
       apiVersion: security.istio.io/v1beta1
       kind: PeerAuthentication
       metadata:
        name: default
        namespace: your-namespace
       spec:
         mtls:
           mode: PERMISSIVE
       ```
      * KServe's default security context restricts containers from running as the root user. This can be achieved by modifying securityContext in the InferenceService yaml. To allow a container to run as root:
      ```yaml
      apiVersion: "serving.kserve.io/v1beta1"
      kind: "InferenceService"
      metadata:
        name: "sklearn-v2-iris"
      spec:
        predictor:
          model:
            modelFormat:
              name: sklearn
            protocolVersion: v2
            runtime: kserve-sklearnserver
            storageUri: "gs://kfserving-examples/models/sklearn/1.0/model"
            securityContext:
              runAsUser: 0
              allowPrivilegeEscalation: true
      ```
      * By default, KServe sets default resource limits for containers to ensure fair resource allocation. This can be overridden by setting resource limits and requests in the Custom Resource yaml. For example,
      ```yaml
      apiVersion: serving.kserve.io/v1beta1
      kind: InferenceService
      metadata:
        name: transformer-collocation
      spec:
        predictor:
          model:
            modelFormat:
              name: pytorch
            storageUri: gs://kfserving-examples/models/torchserve/image_classifier/v1
            resources:
              requests:
                cpu: 100m
                memory: 256Mi
              limits:
                cpu: 1
                memory: 1Gi
      ```
      * KServe disables automatic default service account injection model server pods as usually they don't require privileges to communicate with Kubernetes API server or other Kubernetes related resources. But users can provide their own ServiceAccount using the InferenceService yaml.
     ```yaml
      apiVersion: serving.kserve.io/v1beta1
      kind: InferenceService
      metadata:
        name: transformer-collocation
      spec:
        predictor:
          model:
            modelFormat:
              name: pytorch
            storageUri: gs://kfserving-examples/models/torchserve/image_classifier/v1
            resources:
              requests:
                cpu: 100m
                memory: 256Mi
              limits:
                cpu: 1
                memory: 1Gi
          serviceAccountName: example-serviceaccount
      ```
  * Security Hygiene  
    * Please describe the frameworks, practices and procedures the project uses to maintain the basic health and security of the project.   
      * *Robust CI/CD Pipeline*: Automated unit, integration, and end-to-end tests are integrated into the CI/CD pipeline to catch bugs early in the pull request (PR) stage.
      * *Vulnerability Scanning*: Weekly scheduled image scanning using Snyk, Go source code scanning via gosec, and automated dependency vulnerability checks with Dependabot help identify and remediate known security issues.
      * *Static Code Analysis*: Code linting and static analysis tools are employed to enforce code quality and detect potential issues before they reach production.
      * *Dependency Management*: The project uses lock files (e.g., pyproject.toml, go.mod) for reproducibility. Automated generation of Software Bill of Materials (SBOMs) is now integrated into the CI/CD pipeline for all container images and components, supporting supply chain security and transparency.
      * *Secure Defaults*: Default Kubernetes manifests and Helm charts are configured with security best practices, such as running containers as non-root and enforcing least privilege.
      * *Code Review Process*: All code changes require peer review and explicit approval before being merged into the main branch.
      * *Secure Image Builds*: Image builds are validated through CI/CD checks to ensure they are secure, reproducible, and consistent.
      * *Transparent Governance*: The project practices open governance with regular public meetings and community channels for raising and discussing security or health concerns.
      * *Structured Releases*: Releases are managed with clear versioning and changelogs, ensuring traceability and transparency for all changes.
    * Describe how the project has evaluated which features will be a security risk to users if they are not maintained by the project?  
      * The project identifies potential risks by analyzing the attack surface, especially for features that interact with user-supplied code, custom containers, user supplied models or external data sources.
      * Features are discussed and reviewed in the community, with security implications considered during design and code review processes.
      * The project tracks the security health of dependencies and evaluates the impact of vulnerabilities or unmaintained components.
  * Cloud Native Threat Modeling  
    * Explain the least minimal privileges required by the project and reasons for additional privileges.  
      * Since KServe is installed cluster-wide (not namespace-scoped), it will need broader RBAC permissions to manage resources across namespaces. 
      * The KServe controller requires permission to watch and reconcile KServe-specific custom resources like InferenceService and to manage associated Kubernetes objects like Deployments and Services. These permissions require cluster-wide privileges.
      * Model serving pods run as non-root by default, without requiring host access or privileged escalation. The default service account injection is disabled by default as these pods do not need to communicate with the Kubernetes API Server. The service account can be explicitly specified if they require access to ConfigMaps or Secrets only when required.
      * Default KServe controller containers run as non-root users, minimizing the risk of privilege escalation.
      * If users deploy custom containers or runtimes, those may require additional permissions (e.g., access to GPUs, external storage, or specific system capabilities).
      * Integrations with external systems (like cloud storage, databases, or monitoring tools) may require additional secrets or service account permissions.
      * If users provide credentials for external model storage systems (e.g., S3 or GCS), KServe controller will need permission to access cluster wide Kubernetes Secrets.
    * Describe how the project is handling certificate rotation and mitigates any issues with certificates.  
      * KServe uses cert-manager to manage TLS certificates required for secure communication, particularly for its validating and mutating admission webhooks. Cert-manager handles the issuance, renewal, and rotation of these certificates automatically without downtime.
      * When downloading models from secure cloud storage (e.g., AWS S3, GCS, Azure Blob), KServe allows mounting custom TLS certificates into the model serving pods. These certificates are typically stored in Kubernetes ConfigMaps or Secrets, and are manually provisioned and rotated by cluster administrators. The serving containers are configured to read these certificates from mounted volumes to establish secure HTTPS connections with the external storage provider.
 
    * Describe how the project is following and implementing [secure software supply chain best practices](https://project.linuxfoundation.org/hubfs/CNCF\_SSCP\_v1.pdf) 
      * *Automated Testing and CI/CD Integration*: KServe integrates automated unit, integration, and end-to-end tests within its CI/CD pipeline. This ensures that any changes are validated early, reducing the risk of introducing vulnerabilities. 
      * *Vulnerability Scanning*: The project employs tools like integrates vulnerability scanning tools such as Snyk, dependabot and gosec to identify and address known security issues in dependencies.
      * *Use of Lock Files and SBOM Generation*: KServe utilizes dependency lock files (e.g., pyproject.toml, go.mod) to ensure reproducibility. Automated generation of Software Bill of Materials (SBOMs) is now integrated into the CI/CD pipeline for all container images and components. This enhances transparency of software components and supports supply chain security.
      * *DCO Check*: Committers are required to sign and comply with the Developer Certificate of Origin (DCO) to affirm the legitimacy and authorship of their contributions.
      * *Branch Protection*: The project enforces branch protection rules to prevent unauthorized changes, enforce status checks, require pull request reviews, and control who can push to protected branches.
      * *Prevent Secrets in Source Code*: Proactively prevents committing secrets to the source code by using tools and GitHub workflows that detect sensitive data in PRs.
      * *License compliance*: Automated license compliance checks are now integrated into the CI/CD pipeline. All dependencies are scanned and validated for license compatibility as part of every pull request and release build, ensuring transparency and legal compliance.
      * *Peer Review*: Commits and builds are validated through peer-reviewed pull request workflows, requiring approval before merge.

## Day 1 - Installation and Deployment Phase

### Project Installation and Configuration

  * Describe what project installation and configuration look like.
      * Please see the installation section in Day 0 section.
      * Customizations around installations are available depending on the adopter's use cases: [Serverless](https://kserve.github.io/website/latest/admin/serverless/serverless/), [ModelMesh](https://kserve.github.io/website/latest/admin/modelmesh/), [raw K8s Deployment](https://kserve.github.io/website/latest/admin/kubernetes_deployment/), [Gateway API](https://kserve.github.io/website/latest/admin/gatewayapi_migration/), and [Envoy AI Gateway](https://kserve.github.io/website/latest/admin/ai-gateway_integration/).
      * Installed KServe controller can be further customized and configured via modifying the `inferenceservice-config` configmap. For example, to disable the top level virtual service, add the flag `"disableIstioVirtualHost": true` under the ingress config in inferenceservice configmap like the following:
      ```
      kubectl edit configmap/inferenceservice-config --namespace kserve
      
      ingress : |- {
        "disableIstioVirtualHost": true
      }
      ```

### Project Enablement and Rollback

  * How can this project be enabled or disabled in a live cluster? Please describe any downtime required of the control plane or nodes.  
      * Set the replica count to 0 in the KServe controller deployment.
      * Existing model server deployments will not be impacted since the KServe controller is scaled down to 0 and would not reconcile the existing deployments or inference services.
  * Describe how enabling the project changes any default behavior of the cluster or running workloads.  
      * Enabling the project again by setting the replica to 1.
      * Running model serving deployments and inference services would be reconciled by the KServe controller again so any changes to existing services would be picked up and reflected in the cluster as the controller starts reconciliation loops.
  * Describe how the project tests enablement and disablement.  
      * No tests exist on this as of today but the behavior should be straightforward as outlined above and have been tested in production environment for a number of adopters.
  * How does the project clean up any resources created, including CRDs?
      * The KServe controller can be deleted via `kubectl delete --server-side -f https://github.com/kserve/kserve/releases/download/v0.15.0/kserve.yaml` and KServe CRDs can be deleted via `kubectl delete --server-side -f https://github.com/kserve/kserve/releases/download/v0.15.0/kserve-cluster-resources.yaml`
      * CRs can be created by KServe controller reconciles with InferenceService objects as their owner references. When these InferenceService objects are deleted, any associated CRs will be deleted as well. All InferenceServices can be deleted via `kubectl delete isvc --all-namespaces`.

### Rollout, Upgrade and Rollback Planning

  * How does the project intend to provide and maintain compatibility with infrastructure and orchestration management tools like Kubernetes and with what frequency? 
      * KServe publishes the list of dependencies with their supported versions for every release. The supported versions are evaluated and upgraded on every official release. 

  * Describe how the project handles rollback procedures.   
      * Upgrade of the Inference service will not be complete unless newer version is active. If newer revision is not active within a configurable amount of time, revision is marked as Failed and traffic will still remain on the old version. The traffic will switch from old to new revision only when the newer version is active.
  * How can a rollout or rollback fail? Describe any impact to already running workloads.  
      * Newer revision will not be active for many reasons including insufficient cluster resources for the newer spec, invalid configuration spec. Traffic will not be switched to the new unless the newer revision is ready to accept traffic. Hence, the already running workloads will not be affected in any case if rollout fails. 
  * Describe any specific metrics that should inform a rollback.  
      * InferenceService status provides the status of the inference whether the endpoint is active. Also, various prometheus metrics are exposed to indicate the status of live traffic. eg: Requests throughput and latency 
  * Explain how upgrades and rollbacks were tested and how the upgrade-\>downgrade-\>upgrade path was tested.  
  * Explain how the project informs users of deprecations and removals of features and APIs.  
      * All API changes are backward compatible which are announced in release notes and in the official website.
    
  * Explain how the project permits utilization of alpha and beta capabilities as part of a rollout.
      * We provide conversion webhooks when CRD versions are updated.


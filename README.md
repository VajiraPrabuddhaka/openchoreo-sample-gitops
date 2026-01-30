# OpenChoreo Sample GitOps Repository

A sample GitOps repository demonstrating how to manage application deployments and platform infrastructure using [OpenChoreo](https://openchoreo.dev) and [Flux CD](https://fluxcd.io/).

This repository serves as a reference for structuring an OpenChoreo platform configuration with multi-environment promotion, reusable component types, and enterprise features like API management and observability.

## Repository Structure

```
.
├── flux/                               # Flux CD GitOps configuration
│   ├── gitrepository.yaml              # Git source definition
│   ├── platform-kustomization.yaml     # Platform infrastructure sync
│   └── projects-kustomization.yaml     # Application projects sync
│
├── platform/                           # Platform-level infrastructure
│   ├── component-types/                # Reusable workload templates
│   │   ├── service.yaml                # Microservice (Deployment + Service + HTTPRoute)
│   │   ├── webapp.yaml                 # Web application (Deployment + HTTPRoute)
│   │   └── scheduled-task.yaml         # Scheduled workload (CronJob)
│   │
│   ├── infra/                          # Infrastructure definitions
│   │   ├── buildplanes/                # Build plane configuration
│   │   ├── dataplanes/                 # Data plane and gateway settings
│   │   ├── deployment-pipelines/       # Environment promotion pipelines
│   │   └── environments/               # Environment definitions (dev/staging/prod)
│   │
│   ├── traits/                         # Cross-cutting concerns
│   │   ├── api-management.yaml         # Expose services as managed APIs
│   │   └── observability-alert-rule.yaml # Alerting and monitoring rules
│   │
│   └── secrets/                        # Secret and certificate configuration
│       └── ca-certificates/
│
└── projects/                           # Application projects
    └── demo-project-gitops/            # Sample project
        ├── project.yaml
        └── components/
            ├── reading-list-service-gitops/   # Go service (Buildpacks)
            ├── react-starter-gitops/          # React web app
            └── greeter-service-gitops/        # Go service (Docker + GitOps)
```

## Key Concepts

### Flux CD Integration

Flux CD watches this repository and reconciles the cluster state to match the declared configuration. The sync is split into two stages:

1. **Platform** (`flux/platform-kustomization.yaml`) — deploys infrastructure resources first (component types, environments, pipelines, data planes).
2. **Projects** (`flux/projects-kustomization.yaml`) — deploys application components after the platform is ready.

### Environments and Deployment Pipeline

Three environments are defined with a promotion pipeline:

```
Development  →  Staging  →  Production
               (auto)      (requires approval)
```

Each environment has its own DNS prefix and is backed by the default data plane.

### Component Types

Component types are reusable templates that generate Kubernetes resources from developer-provided parameters. Three types are included:

| Type | Workload | Supported Build Workflows |
|------|----------|--------------------------|
| **Service** | Deployment + Service + HTTPRoute | Google Cloud Buildpacks, Ballerina, Docker |
| **Web Application** | Deployment + HTTPRoute | React |
| **Scheduled Task** | CronJob | Google Cloud Buildpacks, Docker |

Component types use template expressions (`${parameters.*}`, `${metadata.*}`, etc.) to generate Kubernetes manifests with configurable replicas, ports, resource limits, environment variables, and secrets.

### Traits

Traits add cross-cutting capabilities to components:

- **API Management** — Exposes a service as a managed REST API with configurable operations, context paths, and authentication policies.
- **Observability Alert Rule** — Configures log-based or metric-based alert rules with severity levels and notification channels.

## Sample Components

The `demo-project-gitops` project includes three sample components sourced from [openchoreo/sample-workloads](https://github.com/openchoreo/sample-workloads):

| Component | Language | Build Workflow | Description |
|-----------|----------|---------------|-------------|
| `reading-list-service-gitops` | Go | Google Cloud Buildpacks | Backend REST service on port 8080 |
| `react-starter-gitops` | React | React | Frontend web application on port 80 |
| `greeter-service-gitops` | Go | Docker (GitOps) | Service using Docker build with GitOps-based deployment |

## Getting Started

### Prerequisites

- A Kubernetes cluster with [OpenChoreo](https://openchoreo.dev) installed
- [Flux CD](https://fluxcd.io/) bootstrapped on the cluster

### Usage

1. **Fork or clone** this repository.

2. **Update the Git source** in `flux/gitrepository.yaml` to point to your repository:
   ```yaml
   spec:
     url: https://github.com/<your-org>/<your-repo>
   ```

3. **Update the GitOps configuration** in `projects/demo-project-gitops/components/greeter-service-gitops/component.yaml` to point to your GitOps deployment repository (look for the `# CHANGE THIS` comment).

4. **Apply the Flux resources** to your cluster:
   ```bash
   kubectl apply -f flux/
   ```

5. Flux will automatically sync the platform infrastructure and project components.

### Customization

- **Add a new environment** — Create a YAML file under `platform/infra/environments/` and add it to the deployment pipeline.
- **Add a new component type** — Define a new template under `platform/component-types/`.
- **Add a new project** — Create a directory under `projects/` with a `project.yaml` and a `components/` subdirectory.
- **Add a new component** — Create a `component.yaml` under your project's `components/` directory referencing a component type.
- **Attach a trait** — Reference a trait in your component configuration to add API management or observability.

## Technology Stack

- **[OpenChoreo](https://openchoreo.dev)** — Platform engineering framework (Custom Resource Definitions)
- **[Flux CD](https://fluxcd.io/)** — GitOps continuous delivery
- **[Kubernetes Gateway API](https://gateway-api.sigs.k8s.io/)** — Traffic routing and ingress
- **[External Secrets Operator](https://external-secrets.io/)** — Secure secret management
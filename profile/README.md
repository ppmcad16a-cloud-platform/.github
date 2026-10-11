# MERN E-Commerce Cloud Platform

A polyglot e-commerce platform organized as independently deployable services with shared authentication, MongoDB persistence, and centralized CI/CD workflows.

## Architecture

```mermaid
flowchart LR
    Client[Client application\nReact] --> Gateway[API gateway\nNode.js]
    Gateway --> Identity[Identity Service\nJava 21 + Spring Boot]
    Gateway --> Catalog[Catalog Service\nNode.js + Express]
    Gateway --> Commerce[Commerce Service\nPython + GraphQL]
    Gateway --> Notification[Notification Service\nNode.js + Socket.IO]
    Commerce -->|order e-mail| Notification
    Identity --> Mongo[(MongoDB Atlas)]
    Catalog --> Mongo
    Commerce --> Mongo
    Notification --> Mongo
    Identity -->|Issues JWT| Client
```

## Implemented Services

### Identity Server

The Java 21 Spring Boot service provides:

- User registration and login
- Password reset and password changes
- User profiles and address management
- Stateless JWT authentication with Spring Security
- BCrypt password hashing
- MongoDB persistence through Spring Data MongoDB

### Catalog Service

The Node.js and Express service provides:

- Product, category, and brand management
- Product reviews and user wishlists
- Filtering, pagination, slugs, and activation state
- JWT and role-based authorization through Passport-JWT
- MongoDB persistence through Mongoose

Both services are independently runnable and use a shared MongoDB data model. The Identity Server issues seven-day HS256 JWTs; Catalog verifies the shared `JWT_SECRET` and uses the roles `ROLE ADMIN`, `ROLE MEMBER`, and `ROLE MERCHANT`.

## Repositories

| Repository | Responsibility | Technology or focus |
|---|---|---|
| [client-service](https://github.com/ppmcad16a-cloud-platform/client-service) | Client application | Frontend boundary |
| [api-gateway-service](https://github.com/ppmcad16a-cloud-platform/api-gateway-service) | API gateway | External routing boundary |
| [identity-service](https://github.com/ppmcad16a-cloud-platform/identity-service) | Authentication, users, addresses | Java 21, Spring Boot, Spring Security, MongoDB |
| [catalog-service](https://github.com/ppmcad16a-cloud-platform/catalog-service) | Products, categories, brands, reviews, wishlists | Node.js, Express, Mongoose, MongoDB |
| [commerce-service](https://github.com/ppmcad16a-cloud-platform/commerce-service) | Cart and orders (GraphQL) | Python 3.12, Flask, Graphene, PyMongo |
| [notification-service](https://github.com/ppmcad16a-cloud-platform/notification-service) | Contact form, order e-mails, support chat | Node.js, Express, Socket.IO |
| [seed-db-service](https://github.com/ppmcad16a-cloud-platform/seed-db-service) | Database initialization | One-off Node.js job |
| [core-cloud-platform](https://github.com/ppmcad16a-cloud-platform/core-cloud-platform) | Shared CI/CD workflows + learning guides | Reusable GitHub Actions |
| [terraform-modules](https://github.com/ppmcad16a-cloud-platform/terraform-modules) | Reusable infrastructure modules | Terraform |
| [aws-platform](https://github.com/ppmcad16a-cloud-platform/aws-platform) | AWS infrastructure (EKS, ECR, Secrets Manager) | Terraform |
| [azure-platform](https://github.com/ppmcad16a-cloud-platform/azure-platform) | Azure infrastructure (AKS, ACR, Key Vault) | Terraform |
| [gcp-platform](https://github.com/ppmcad16a-cloud-platform/gcp-platform) | Google Cloud infrastructure (GKE, Artifact Registry, Secret Manager) | Terraform |
| [global-platform](https://github.com/ppmcad16a-cloud-platform/global-platform) | Cloudflare DNS / load balancing, MongoDB Atlas access | Terraform |
| [platform-deployments](https://github.com/ppmcad16a-cloud-platform/platform-deployments) | Helm chart, cluster add-ons, Jenkins pipelines | Helm, Jenkins |
| [security-platform](https://github.com/ppmcad16a-cloud-platform/security-platform) | Secret stores, network policies, least-privilege docs | Kubernetes, IAM |
| [observability-platform](https://github.com/ppmcad16a-cloud-platform/observability-platform) | Metrics, logs, dashboards, alerts, cost | Prometheus, Grafana, Loki |

## Reusable CI/CD

The [`core-cloud-platform`](https://github.com/ppmcad16a-cloud-platform/core-cloud-platform) repository centralizes reusable GitHub Actions workflows. Application repositories provide their own triggers and call the shared workflows with service-specific inputs.

```mermaid
flowchart TB
    NodeSvc[gateway, catalog, notification,\nclient, seed-db] -->|workflow_call @v1| Node[node-app.yaml]
    Identity[identity-service] -->|workflow_call @v1| Java[java-app.yaml]
    Commerce[commerce-service] -->|workflow_call @v1| Py[python-app.yaml]
    Node & Java & Py -->|optional| Docker[docker-build.yaml\nbuild + Trivy gate]
    NodeSvc & Identity & Commerce --> Sec[security-scan.yaml\ngitleaks]
```

Current reusable workflows provide:

- Node.js 22, Java 21 (single `mvn verify`) and Python 3.12 build + test, with each repo's coverage gate
- Container image build with a Trivy vulnerability gate (fixable HIGH/CRITICAL fail the build)
- Secret scanning of the full git history with gitleaks
- Every action pinned to a commit SHA, least-privilege `permissions`, scanners checksum-verified
- Thin service-level callers that reference the tested `@v1` release tag

The services also use focused automated testing:

- Catalog: Jest, Supertest, and MongoDB Memory Server
- Identity: JUnit, Spring Boot Test, Spring Security Test, and JaCoCo

## Platform Roadmap

The platform is being extended toward a multi-cloud delivery model covering AWS, Microsoft Azure, and Google Cloud, with Kubernetes as the common runtime.

Planned capabilities include:

- Docker image builds and registry publishing
- Terraform provisioning and remote state
- Kubernetes deployment workflows
- Source, dependency, IaC, secret, and container scanning
- Observability dashboards, metrics, logs, traces, and alerts
- Cloud secret-manager integration through OIDC
- Staging and production environments
- Approval-based promotion, health checks, and rollback

See the [core-cloud-platform README](https://github.com/ppmcad16a-cloud-platform/core-cloud-platform#readme) for reusable workflow standards and implementation details.

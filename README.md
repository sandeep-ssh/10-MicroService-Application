# 🛒 Cloud-Native Microservices on Azure AKS (ARM64) — End-to-End DevSecOps Pipeline

> Deploying Google's Online Boutique (10 microservices) on Azure Kubernetes Service with ARM64 nodes, a fully automated Azure Pipelines CI/CD pipeline, SonarQube static analysis, and Azure Container Registry — built entirely from scratch on a self-hosted agent.

---

## 📌 Project Overview

This project demonstrates the end-to-end deployment of **Google's Online Boutique**, a cloud-native microservices e-commerce application, onto **Azure Kubernetes Service (AKS)** running **ARM64 nodes**. The entire build, test, and deployment lifecycle is automated via **Azure Pipelines Classic Editor** using a self-hosted build agent running on an **AWS EC2 Ubuntu instance**.

| Attribute | Detail |
|-----------|--------|
| **Application** | Google Online Boutique (10 microservices) |
| **Container Registry** | Azure Container Registry (ACR) — `sandeepk8.azurecr.io` |
| **Kubernetes Cluster** | AKS — `Sandeep-k8` (ARM64 node pool) |
| **CI/CD Platform** | Azure Pipelines Classic Editor |
| **Build Agent** | Self-hosted on AWS EC2 (Ubuntu, ARM64-capable via QEMU) |
| **Code Quality** | SonarQube Community Edition (containerised) |
| **Languages** | Go, Python, Java, Node.js, C# (.NET) |
| **Infrastructure** | Azure (AKS, ACR), AWS (EC2 agent) |

---

## 🏗️ Architecture Overview

![Azure Architecture Diagram](./assets/azurecicd.png)


```
┌─────────────────────────────────────────────────────────────────┐
│                        Azure DevOps                             │
│  ┌─────────────┐    ┌──────────────────────────────────────┐   │
│  │ Azure Repos │───▶│   Azure Pipelines (Classic Editor)   │   │
│  │    (Git)    │    │  - SonarQube Analysis                │   │
│  └─────────────┘    │  - Docker buildx (ARM64)             │   │
│                     │  - Push to ACR                        │   │
│                     │  - kubectl apply                      │   │
│                     └──────────────┬───────────────────────┘   │
└──────────────────────────────────  │  ──────────────────────────┘
                                     │ runs on
                    ┌────────────────▼──────────────────┐
                    │       AWS EC2 (Self-Hosted Agent)  │
                    │  - Azure Pipelines Agent           │
                    │  - Docker CE + buildx              │
                    │  - QEMU binfmt (ARM64 emulation)   │
                    │  - SonarQube container (port 9000) │
                    └────────────────┬──────────────────┘
                                     │ push images
                    ┌────────────────▼──────────────────┐
                    │   Azure Container Registry (ACR)  │
                    │   sandeepk8.azurecr.io            │
                    └────────────────┬──────────────────┘
                                     │ pull images
                    ┌────────────────▼──────────────────┐
                    │     Azure Kubernetes Service       │
                    │     (AKS — ARM64 node pool)        │
                    │                                    │
                    │  ┌──────────┐  ┌───────────────┐  │
                    │  │ frontend │  │ checkoutservice│  │
                    │  ├──────────┤  ├───────────────┤  │
                    │  │ cart     │  │ payment        │  │
                    │  ├──────────┤  ├───────────────┤  │
                    │  │ product  │  │ recommendation │  │
                    │  ├──────────┤  ├───────────────┤  │
                    │  │ currency │  │ shipping       │  │
                    │  ├──────────┤  ├───────────────┤  │
                    │  │ email    │  │ adservice      │  │
                    │  ├──────────┤  ├───────────────┤  │
                    │  │  redis   │  │ loadgenerator  │  │
                    │  └──────────┘  └───────────────┘  │
                    └───────────────────────────────────┘
```

### Microservices Map

| Service | Language | Port | Role |
|---------|----------|------|------|
| frontend | Go | 8080 | Web UI, routes to all backends |
| cartservice | C# (.NET) | 7070 | Shopping cart via Redis |
| productcatalogservice | Go | 3550 | Product listings |
| currencyservice | Node.js | 7000 | Currency conversion |
| paymentservice | Node.js | 50051 | Payment processing |
| shippingservice | Go | 50051 | Shipping quotes |
| emailservice | Python | 8080 | Order confirmation emails |
| checkoutservice | Go | 5050 | Orchestrates checkout flow |
| recommendationservice | Python | 8080 | Product recommendations |
| adservice | Java | 9555 | Ad targeting |
| loadgenerator | Python (Locust) | — | Simulates user traffic |
| redis-cart | Redis | 6379 | Cart persistence |

---

## ☁️ Azure Well-Architected Framework Alignment

| Pillar | Implementation |
|--------|---------------|
| **Reliability** | Kubernetes liveness & readiness probes with `initialDelaySeconds` on all services; AKS node auto-healing; Redis persistence via emptyDir volume |
| **Security** | ACR integrated with AKS via managed identity (`az aks update --attach-acr`); SonarQube SAST on every pipeline run; `readOnlyRootFilesystem`, dropped capabilities, and non-root users enforced via pod `securityContext` |
| **Cost Optimisation** | ARM64 nodes consume ~40% less power than x86 equivalents; self-hosted agent avoids Microsoft-hosted agent costs; right-sized CPU/memory requests and limits per service |
| **Operational Excellence** | Fully automated CI/CD pipeline; SonarQube quality gates; structured JSON logging in all services; Azure Pipelines audit trail |
| **Performance Efficiency** | ARM64 architecture for better price/performance ratio; Docker layer caching for faster builds; multi-stage Dockerfiles minimise image sizes |

---

## ⚠️ Challenges Faced

| # | Challenge | Root Cause | Resolution |
|---|-----------|------------|------------|
| 1 | Docker buildx unavailable on agent | Distro `docker.io` package lacks buildx plugin | Reinstalled using official Docker CE + `docker-buildx-plugin` |
| 2 | iptables conflict after Docker install | Ubuntu uses nftables; Docker needs legacy iptables | `update-alternatives --set iptables /usr/sbin/iptables-legacy` |
| 3 | ARM64 cross-compilation failing | QEMU binfmt not installed | Installed `tonistiigi/binfmt`, created `arm64builder` with docker-container driver |
| 4 | `--platform` pin in all Dockerfiles | Hardcoded `--platform=linux/arm64` in FROM lines conflicts with buildx | Removed platform pins from all 10 Dockerfiles |
| 5 | `google-cloud-profiler` failing on ARM64 | No ARM64 binary; requires `g++` to compile C extension | Filtered profiler from requirements with `grep -v`; added build tools to builder stage |
| 6 | `pprof` native module failing (Node.js) | No ARM64 prebuilt binary for pprof | Added `--ignore-scripts` to `npm ci` |
| 7 | shippingservice Dockerfile written as Rust | Mismatch between Dockerfile language and actual Go source | Rewrote as Go multi-stage build; verified from `go.mod` and `main.go` |
| 8 | `grpc_health_probe` missing in all images | Not included in base images | Added ARM64 binary download to every service's runtime stage |
| 9 | Stale buildx builder container | buildx builder container silently died | Deleted and recreated with `docker buildx rm` + `docker buildx create` |
| 10 | ImagePullBackOff on AKS | AKS not authorised to pull from ACR | `az aks update --attach-acr sandeepk8` |
| 11 | Wrong image registry in all deployment YAMLs | YAMLs pointed to `sandeepssh/` instead of `sandeepk8.azurecr.io/` | Updated all image references across deployment YAMLs |
| 12 | adservice CrashLoopBackOff | GCP Profiler JVM agent referenced in `build.gradle` | Removed `defaultJvmOpts` from both Gradle tasks |
| 13 | recommendationservice Python version mismatch | Builder copied python3.10 libs from python3.11 image | Aligned all stages to consistent python3.11 base |
| 14 | loadgenerator ENTRYPOINT broken | JSON array ENTRYPOINT blocked env var expansion | Changed to shell form ENTRYPOINT |
| 15 | shippingservice image had `/bin/sh` as entrypoint | Stale image in ACR from before ENTRYPOINT fix | Force rebuilt with `--no-cache` and re-pushed |
| 16 | All services crashing due to probes firing at `delay=0s` | No `initialDelaySeconds` on liveness/readiness probes | Added `initialDelaySeconds: 10–20` to all probes in deployment YAML |
| 17 | YAML corruption after automated edits | Python regex inserted `initialDelaySeconds` at wrong indent level | Validated with `yaml.safe_load_all()`; fixed manually in VS Code |
| 18 | EC2 agent IP changes on restart | No Elastic IP assigned | Documented restart procedure; recommended assigning Elastic IP |
| 19 | `az` CLI missing on agent for ACR login | Azure CLI not installed on EC2 | Used `docker login` with credentials from `az acr credential show` on Mac |
| 20 | Two shippingservice ReplicaSets running | Old ReplicaSets from failed rollouts not cleaned up | Deleted stale ReplicaSet with `kubectl delete replicaset` |

---

## ⚖️ Architectural Trade-offs & Non-Goals

### Trade-offs Made

| Decision | Chosen Approach | Alternative Considered | Reason |
|----------|----------------|----------------------|--------|
| Build agent location | AWS EC2 (self-hosted) | Azure-hosted agents | ARM64 cross-build capability; cost control |
| Container registry | Azure ACR | Docker Hub | Native AKS integration via managed identity |
| Kubernetes probes | `initialDelaySeconds` + exec probes | Startup probes | Simpler; sufficient for this workload |
| Profiler exclusion | Skip `google-cloud-profiler` entirely | Stub/mock the profiler | Not running on GCP; profiler has no ARM64 support |
| Image build strategy | `docker buildx` with QEMU emulation | Native ARM64 build machine | Cost; agent already available |

### Non-Goals
- **High Availability**: Single node pool, no multi-region replication — this is a portfolio/learning project
- **Production secrets management**: No Azure Key Vault or Kubernetes secrets encryption at rest
- **GitOps**: No ArgoCD or Flux — pipeline applies manifests directly via `kubectl`
- **Service mesh**: No Istio/Linkerd — inter-service communication is plain gRPC/HTTP
- **Horizontal Pod Autoscaling**: Static replica counts; no HPA configured

---

## 💡 Why This Project Matters

Most Kubernetes tutorials use pre-built Docker Hub images and managed pipelines. This project is different:

- **Everything is built from scratch** — all 10 microservices are compiled and containerised as part of the pipeline
- **ARM64 is non-trivial** — cross-platform builds, architecture-incompatible dependencies, and missing binaries are real-world problems that required deep debugging
- **Multi-language complexity** — Go, Python, Java, Node.js, and C# each have unique ARM64 pitfalls
- **Self-hosted infrastructure** — operating a build agent across two cloud providers (Azure + AWS) mirrors real enterprise constraints
- **End-to-end ownership** — from `git push` to a live URL, every component is understood and controlled

---

## 📋 Architectural Decision Records (ADRs)

| ADR | Decision | Context | Consequences |
|-----|----------|---------|--------------|
| ADR-001 | Use ARM64 AKS nodes | Better price/performance ratio for sustained workloads | Required QEMU emulation on agent; ARM64-incompatible dependencies had to be resolved |
| ADR-002 | Self-hosted agent on AWS EC2 | Need ARM64 build capability; Microsoft-hosted agents are x86 only | Agent IP changes on restart; manual restart procedure required |
| ADR-003 | Use `docker buildx` with docker-container driver | Enables multi-platform builds from x86 agent | Builder container can die silently; requires recreation after agent restart |
| ADR-004 | Strip `google-cloud-profiler` from Python services | Package has no ARM64 binary and requires GCP environment | Profiling unavailable; acceptable as this is not a GCP deployment |
| ADR-005 | Use `--ignore-scripts` for Node.js currencyservice | `pprof` native module has no ARM64 prebuilt binary | Profiling disabled; no functional impact on currency conversion |
| ADR-006 | Add `initialDelaySeconds` to all Kubernetes probes | gRPC services need startup time before health checks begin | Slightly slower pod readiness detection; prevents false-positive probe failures |
| ADR-007 | Attach ACR to AKS via managed identity | Eliminates need for image pull secrets in pod specs | AKS service principal must have AcrPull role; one-time setup required |
| ADR-008 | Use multi-stage Dockerfiles for all services | Separates build dependencies from runtime image | Smaller, more secure runtime images; build tools not present in production containers |
| ADR-009 | Run SonarQube on self-hosted agent | Avoids SonarCloud subscription cost; keeps analysis local | SonarQube container must be manually started after agent restart |
| ADR-010 | Use Classic Editor pipeline (not YAML) | Easier visualisation and step-by-step debugging for learning | Less portable than YAML pipelines; harder to version control pipeline definition |

---

## 🚀 Next Improvements

| Priority | Improvement | Benefit |
|----------|------------|---------|
| High | Assign Elastic IP to EC2 agent | Eliminates manual SonarQube URL update on every restart |
| High | Convert pipeline to YAML (`azure-pipelines.yml`) | Pipeline definition version-controlled alongside code |
| High | Add Azure Key Vault for secrets | Eliminate plaintext ACR credentials in pipeline variables |
| Medium | Add Horizontal Pod Autoscaler (HPA) | Auto-scale services under load from loadgenerator |
| Medium | Enable ArgoCD or Flux for GitOps | Declarative, auditable deployments; drift detection |
| Medium | Add container vulnerability scanning (Trivy or Defender for Containers) | Catch CVEs in base images before deployment |
| Medium | Use Azure Monitor + Prometheus for observability | Real-time metrics, alerting, and dashboards |
| Low | Replace QEMU emulation with native ARM64 build agent | Faster builds; eliminate architecture emulation overhead |
| Low | Add Helm charts for deployment management | Templated, reusable Kubernetes manifests |
| Low | Enable TLS on frontend via cert-manager + Let's Encrypt | Production-grade HTTPS endpoint |

---

## 📚 Lessons Learned

| Area | Lesson |
|------|--------|
| **ARM64** | ARM64 is not just "compile with a different flag" — every dependency must be verified for architecture compatibility, especially native extensions in Python and Node.js |
| **Docker buildx** | The buildx builder is stateful; its container can die silently between pipeline runs. Always verify with `docker buildx inspect` and recreate if needed |
| **Kubernetes probes** | `initialDelaySeconds` is not optional for gRPC services. A probe firing before the service binds to its port will kill the container before it logs anything, making debugging nearly impossible |
| **Dockerfile hygiene** | Multi-stage builds are essential — not just for image size, but for separating build-time dependencies (compilers, dev headers) from runtime images |
| **YAML editing** | Never use automated text replacement on YAML files without validating with a parser (`yaml.safe_load_all`). Indentation errors are invisible to the eye but fatal to `kubectl` |
| **Debugging silent crashes** | When `kubectl logs` returns empty, the container is crashing before writing to stdout. Use `docker run` locally with `--entrypoint /bin/sh` to inspect the image and run the binary manually |
| **Cross-cloud pipelines** | Running a build agent on AWS while deploying to Azure works well — but operational overhead (IP changes, manual restarts) accumulates. Static IPs and systemd service units are worth the investment |
| **GCP dependencies** | Open-source projects designed for GCP (like Online Boutique) embed GCP-specific tooling (Cloud Profiler, Stackdriver) that breaks silently outside GCP. Always audit dependencies for cloud-specific assumptions |

---

## 💼 Business Value Delivered

| Value | Description |
|-------|-------------|
| **Cost Efficiency** | ARM64 nodes reduce compute costs by ~40% vs equivalent x86 instances; self-hosted agent eliminates per-minute pipeline billing |
| **Developer Velocity** | Fully automated pipeline from code commit to running pod; SonarQube catches bugs before deployment |
| **Security Posture** | SAST integrated into every build; least-privilege container security contexts; managed identity eliminates credential sprawl |
| **Operational Resilience** | Kubernetes self-healing with properly tuned health probes; graceful termination periods configured per service |
| **Skills Transferability** | Every pattern used here (multi-stage builds, ARM64 cross-compilation, AKS + ACR integration, probe tuning) applies directly to production enterprise environments |
| **Portfolio Differentiation** | Demonstrates ability to debug complex, multi-layer infrastructure problems across networking, containerisation, orchestration, and CI/CD — not just follow tutorials |

---

## 🪞 Final Reflection

This project started as a straightforward "deploy a sample app to Kubernetes" exercise and evolved into a comprehensive lesson in the real-world complexity of cloud-native infrastructure.

The most valuable experiences were not the successes — they were the failures: the silent crashes with empty logs, the YAML that looked correct but wasn't, the binary that ran perfectly locally but exited with code 0 in Kubernetes, the build agent that silently switched architectures mid-pipeline.

Each of these required methodical debugging: isolating variables, reading source code, inspecting running containers, and understanding the full stack from Dockerfile to pod spec to Kubernetes control plane behaviour.

**The core insight:** In cloud-native engineering, the gap between "it works on my machine" and "it runs reliably in production" is where real skill is built. This project was that gap, end to end.

---

## 🛠️ Tech Stack Summary

![Azure](https://img.shields.io/badge/Azure-AKS%20%7C%20ACR-0078D4?logo=microsoftazure)
![AWS](https://img.shields.io/badge/AWS-EC2%20Build%20Agent-FF9900?logo=amazonaws)
![Docker](https://img.shields.io/badge/Docker-buildx%20ARM64-2496ED?logo=docker)
![Kubernetes](https://img.shields.io/badge/Kubernetes-1.29-326CE5?logo=kubernetes)
![SonarQube](https://img.shields.io/badge/SonarQube-SAST-4E9BCD?logo=sonarqube)
![Go](https://img.shields.io/badge/Go-microservices-00ADD8?logo=go)
![Python](https://img.shields.io/badge/Python-microservices-3776AB?logo=python)
![Java](https://img.shields.io/badge/Java-adservice-ED8B00?logo=openjdk)
![Node.js](https://img.shields.io/badge/Node.js-microservices-339933?logo=nodedotjs)

---

*Built by Sandeep Hegde — connecting infrastructure engineering with real-world cloud deployment challenges.*
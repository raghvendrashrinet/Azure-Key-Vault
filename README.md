## Azure Key Vault 
### Use Cases
 - Centralized Secrets Management via Azure Key Vault
 - Kubernetes Pod Access to Key Vault via Secrets Store CSI Drive
 - CI/CD Pipeline Integration with Azure Key Vault
 - Cross-Resource Authentication via Managed Identity & OIDC Federation

---

### 1. Kubernetes Pod Access to Azure Key Vault (Primary Demo)

**Overview:**

Enables pods running inside an Azure Kubernetes Service (AKS) cluster to dynamically fetch and mount sensitive secrets directly from Azure Key Vault as volume files — without storing them inside native Kubernetes Secrets objects.
```
+---------------+ OIDC / Managed Identity +-----------------+
| AKS Pod | ==============================> | Azure Key Vault |
| (CSI Mounted) | <------------------------------ | (Secrets) |
+---------------+ Secrets Mounted as Files +-----------------+
```

### Key Concepts & Components Used

- **Secrets Store CSI Driver & Azure Provider** – Interfacing layer between Kubernetes storage APIs and external secret management providers. 
- **SecretProviderClass** – Custom Kubernetes resource defining which Key Vault objects (keys/secrets) to fetch. 
- **Workload Identity (OIDC Federation)** – Binds a Kubernetes Service Account to an Azure User-Assigned Managed Identity for passwordless pod authentication.
- **Volume Mounts** – Mounts fetched secrets as read-only files inside the pod filesystem (e.g., `/mnt/secrets-store`). 

---

### 2. Centralized Secrets Management & Compliance Policy

**Overview:**

Consolidates sensitive information—database connection strings, API tokens, passwords, TLS certificates—into a single managed cloud vault instead of dispersing them across multiple tools.
```
                +-------------------+
                | Azure Key Vault   |
                | (Central Hub)     |
                +---------+---------+
                          |
       +------------------+----------------------+
       |                  |                      |
       v                  v                      v
+--------------+   +---------------+      +---------------+
|  AKS Pods    |   |  Virtual      |      | Serverless    |
|              |   |  Machines     |      | Functions     |
+--------------+   +---------------+      +---------------+   
```

### Key Concepts & Components Used

- **Centralized Vault Governance** – Standardizes secret storage and enforcement of organizational security policies.
- **Automated Secret Rotation** – Simplifies compliance rules (e.g., rotating passwords every 90–180 days) across all consuming services without updating individual application code or Kubernetes manifests.
- **Azure RBAC** – Implements fine-grained access policies to control read, write, and delete permissions for specific identities.

---

### 3. Dynamic Secrets Integration in CI/CD Pipelines

**Overview:**

Externalizes pipeline-level credentials and execution keys from CI/CD configurations (e.g., GitHub Actions or Azure Pipelines), retrieving them dynamically during pipeline execution.
```
+--------------------+ Retrieve Secrets +-------------------+
| CI/CD Pipeline | ========================> | Azure Key Vault |
| (GitHub/Azure DevOps) <----------------------- | (Registry Keys, |
+--------------------+ Inject at Runtime | Terraform Keys) |
+-------------------+
```

### Key Concepts & Components Used

- **External Secret Retrieval** – Avoids hardcoding sensitive values or environment variables directly inside source control or pipeline definitions.
- **Infrastructure & Tooling Credentials** – Dynamically fetches Docker registry connection strings, Terraform backend keys, and Ansible vault secrets at runtime.

### 4. Cross-Resource Authentication via Managed Identity & OIDC Federation
Azure Kubernetes Service (AKS) cluster to access external Azure resources (such as Azure Key Vault) securely using User-Assigned Managed Identities and OIDC Workload Identity Federation, eliminating the need for hardcoded client secrets or tokens.
```
┌─────────────────────────────────────────────────────────────┐
│                        AKS Cluster                          │
│                                                             │
│  ┌───────────────────────────┐     ┌─────────────────────┐  │
│  │   Kubernetes Service      │ ─── │    Pod / Container  │  │
│  │   Account (SA)            │     │   Workload / CSI    │  │
│  └─────────────┬─────────────┘     └──────────┬──────────┘  │
└────────────────┼──────────────────────────────┼─────────────┘
                 │ (OIDC Issuer Trust)          │
                 ▼                              │
┌─────────────────────────────────────┐         │
│  Azure AD / Entra ID                │         │
│  ┌───────────────────────────────┐  │         │ (Uses Azure SDK /
│  │ User-Assigned Managed Identity│  │ ◄───────┘  Secret Store CSI)
│  └─────────────┬─────────────────┘  │
└────────────────┼────────────────────┘
                 │ (RBAC Assignment)
                 ▼
┌─────────────────────────────────────┐
│          Azure Key Vault            │
└─────────────────────────────────────┘
```

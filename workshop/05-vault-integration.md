# Vault Integration & Secret Management

## 🎯 Purpose

This document addresses Terraform + Vault integration patterns, helping organizations leverage dynamic secrets and improve the synergy between Terraform and Vault teams.

---

## 🔐 Understanding Vault + Terraform Integration

### **Common Anti-Patterns**

| Anti-Pattern | Why It's Bad | Better Approach |
|--------------|--------------|-----------------|
| **Static secrets in Vault** | Defeats the purpose of Vault | Use dynamic secret engines |
| **Long-lived tokens** | If leaked, exploitable for days/weeks | Short TTL (< 1 hour) |
| **Root/admin tokens in CI/CD** | Massive security risk | AppRole with scoped policies |
| **Secrets in Terraform state** | State file contains plaintext | Use `sensitive` outputs, encrypt state |
| **No token renewal** | Runs fail mid-execution | Enable auto-renewal or use short jobs |

### **Proper Integration Pattern**

```
┌────────────────────────────────────────────────────────────┐
│                    Terraform Execution                      │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  1. Authenticate to Vault (AppRole/JWT/OIDC)               │
│  2. Receive short-lived token (TTL: 15-60 min)             │
│  3. Request dynamic cloud credentials                       │
│  4. Vault generates AWS/Azure/GCP credentials               │
│  5. Terraform uses credentials                              │
│  6. Credentials expire automatically                        │
│  7. Full audit trail in Vault                               │
│                                                             │
└────────────────────────────────────────────────────────────┘
```

---

## 🗣️ Discussion Guide: Current Vault Usage

### **Understanding the Relationship**

**1. "How do Terraform modules currently retrieve secrets from Vault?"**

*Why this matters*: Reveals whether they're using Vault properly (dynamic secrets, short TTLs) or as a static secret store.

**Follow-up probes:**
- "Do you use `vault_generic_secret` or `vault_kv_secret_v2`?" (Static secrets)
- "Do you use dynamic secret engines like `vault_aws_access_credentials`?" (Dynamic - better)
- "How often do secrets rotate?"
- "What's the TTL on retrieved secrets?"

**2. "What's the relationship between the Vault team and the Terraform team?"**

*Why this matters*: Organizational silos often create friction. Shared goals and collaboration reduce friction.

**Assess the relationship:**
- Do they have regular sync meetings?
- Is there shared documentation?
- Do they have joint design reviews?
- Who owns Vault policies for Terraform?
- Are there SLAs for Vault requests?

**3. "Do modules use Vault's dynamic secret engines (AWS, Azure, database credentials) or static secrets?"**

*Why this matters*: Dynamic secrets are the right pattern. Static secrets from Vault defeats the purpose.

**Key distinction:**
- **Static**: Vault stores pre-created credentials (like password manager)
- **Dynamic**: Vault generates credentials on-demand (short-lived, auto-revoked)

**4. "How does Vault authentication work for Terraform runs in your pipelines?"**

*Why this matters*: Checks if they're using secure patterns (AppRole, JWT, Kubernetes auth) vs tokens in environment variables.

| Auth Method | Security Level | Use Case |
|-------------|----------------|----------|
| **OIDC/JWT** | ✅✅ Best | HCP Terraform, GitHub Actions, GitLab CI |
| **Kubernetes** | ✅✅ Best | Terraform running in K8s pods |
| **AppRole** | ✅ Good | CI/CD systems, automation |
| **Token** (short TTL) | ⚠️ Acceptable | Local dev, short-lived |
| **Token** (long TTL) | ❌ Bad | Never in production |
| **Root token** | ❌❌ Terrible | Never, ever |

**5. "Have there been cases where Terraform couldn't access secrets it needed from Vault? What happened?"**

*Why this matters*: Identifies integration pain points.

**Common issues:**
- Token expiration mid-run
- Policy misconfiguration
- Network connectivity
- Secret path changes
- Vault downtime

---

## 🔑 Vault Authentication Patterns

### **Option 1: AppRole (CI/CD Pipelines)**

**How it works:**
- CI/CD system has Role ID (public-ish)
- Secret ID generated per run (short-lived)
- Authenticates to Vault
- Receives token with scoped permissions

**Terraform Configuration:**

```hcl
# Configure Vault provider
provider "vault" {
  address = var.vault_address
  
  auth_login {
    path = "auth/approle/login"
    
    parameters = {
      role_id   = var.vault_role_id
      secret_id = var.vault_secret_id
    }
  }
}

# Retrieve dynamic AWS credentials from Vault
data "vault_aws_access_credentials" "terraform" {
  backend = "aws"
  role    = "terraform-deploy"
  type    = "creds"  # or "sts" for STS tokens
  ttl     = "15m"
}

# Use dynamic credentials in AWS provider
provider "aws" {
  region     = var.aws_region
  access_key = data.vault_aws_access_credentials.terraform.access_key
  secret_key = data.vault_aws_access_credentials.terraform.secret_key
}
```

**Pipeline Configuration (GitHub Actions):**

```yaml
name: Terraform Apply

on:
  push:
    branches: [main]

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Get Vault Secret ID
        id: vault_secret
        run: |
          # Call Vault API to generate Secret ID
          SECRET_ID=$(curl -X POST \
            -H "X-Vault-Token: ${{ secrets.VAULT_APPROLE_TOKEN }}" \
            $VAULT_ADDR/v1/auth/approle/role/terraform/secret-id \
            | jq -r '.data.secret_id')
          echo "::add-mask::$SECRET_ID"
          echo "secret_id=$SECRET_ID" >> $GITHUB_OUTPUT
      
      - name: Terraform Apply
        env:
          VAULT_ROLE_ID: ${{ secrets.VAULT_ROLE_ID }}
          VAULT_SECRET_ID: ${{ steps.vault_secret.outputs.secret_id }}
        run: |
          terraform init
          terraform apply -auto-approve
```

**Vault Policy:**

```hcl
# Policy for Terraform AppRole
path "aws/creds/terraform-deploy" {
  capabilities = ["read"]
}

path "secret/data/terraform/*" {
  capabilities = ["read", "list"]
}
```

---

### **Option 2: JWT/OIDC (GitHub Actions, GitLab CI, HCP Terraform)**

**How it works:**
- CI system generates JWT token (no secrets!)
- Terraform authenticates with JWT
- Vault validates JWT with OIDC provider
- Returns scoped token

**Benefits:**
- ✅ No secrets to manage
- ✅ Cannot be leaked (token bound to job)
- ✅ Industry standard
- ✅ Simplest to implement

**Terraform Configuration:**

```hcl
provider "vault" {
  address = var.vault_address
  
  auth_login_jwt {
    role = "terraform"
    jwt  = var.ci_jwt_token  # Provided by CI system
  }
}
```

**GitHub Actions Configuration:**

```yaml
name: Terraform with OIDC

on:
  push:
    branches: [main]

permissions:
  id-token: write  # Required for OIDC
  contents: read

jobs:
  terraform:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Get OIDC Token
        id: oidc
        run: |
          OIDC_TOKEN=$(curl -H "Authorization: bearer $ACTIONS_ID_TOKEN_REQUEST_TOKEN" \
            "$ACTIONS_ID_TOKEN_REQUEST_URL" | jq -r '.value')
          echo "::add-mask::$OIDC_TOKEN"
          echo "token=$OIDC_TOKEN" >> $GITHUB_OUTPUT
      
      - name: Terraform Apply
        env:
          VAULT_ADDR: ${{ secrets.VAULT_ADDR }}
          CI_JWT_TOKEN: ${{ steps.oidc.outputs.token }}
        run: |
          terraform init
          terraform apply -auto-approve
```

**Vault JWT Auth Configuration:**

```bash
# Enable JWT auth
vault auth enable jwt

# Configure GitHub OIDC
vault write auth/jwt/config \
  bound_issuer="https://token.actions.githubusercontent.com" \
  oidc_discovery_url="https://token.actions.githubusercontent.com"

# Create role for Terraform
vault write auth/jwt/role/terraform \
  role_type="jwt" \
  bound_audiences="https://github.com/my-org" \
  bound_subject="repo:my-org/my-repo:ref:refs/heads/main" \
  user_claim="actor" \
  policies="terraform-deploy" \
  ttl="15m"
```

---

### **Option 3: Kubernetes Auth (Terraform in K8s)**

**For Terraform running as Kubernetes jobs:**

```hcl
provider "vault" {
  address = var.vault_address
  
  auth_login_kubernetes {
    role = "terraform"
    jwt  = file("/var/run/secrets/kubernetes.io/serviceaccount/token")
  }
}
```

**Vault Kubernetes Auth Setup:**

```bash
# Enable Kubernetes auth
vault auth enable kubernetes

# Configure Kubernetes
vault write auth/kubernetes/config \
  kubernetes_host="https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT" \
  kubernetes_ca_cert=@/var/run/secrets/kubernetes.io/serviceaccount/ca.crt

# Create role
vault write auth/kubernetes/role/terraform \
  bound_service_account_names=terraform \
  bound_service_account_namespaces=automation \
  policies=terraform-deploy \
  ttl=1h
```

---

## ☁️ Dynamic Cloud Credentials

### **AWS Dynamic Credentials**

**Vault Setup:**

```bash
# Enable AWS secrets engine
vault secrets enable aws

# Configure AWS root credentials (Vault uses these to generate dynamic creds)
vault write aws/config/root \
  access_key=$AWS_ACCESS_KEY \
  secret_key=$AWS_SECRET_KEY \
  region=us-east-1

# Create Terraform deployment role
vault write aws/roles/terraform-deploy \
  credential_type=iam_user \
  policy_document=-<<EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["ec2:*", "s3:*", "rds:*"],
      "Resource": "*"
    }
  ]
}
EOF
```

**Terraform Usage:**

```hcl
data "vault_aws_access_credentials" "terraform" {
  backend = "aws"
  role    = "terraform-deploy"
  ttl     = "15m"  # Credentials expire in 15 minutes
}

provider "aws" {
  access_key = data.vault_aws_access_credentials.terraform.access_key
  secret_key = data.vault_aws_access_credentials.terraform.secret_key
}

# Deploy resources...
```

**What happens:**
1. Terraform requests credentials from Vault
2. Vault creates new IAM user (or STS token)
3. Vault returns credentials
4. Terraform uses credentials
5. After 15 minutes, Vault revokes/deletes credentials

---

### **Azure Dynamic Credentials**

**Vault Setup:**

```bash
# Enable Azure secrets engine
vault secrets enable azure

# Configure Azure
vault write azure/config \
  subscription_id=$AZURE_SUBSCRIPTION_ID \
  tenant_id=$AZURE_TENANT_ID \
  client_id=$AZURE_CLIENT_ID \
  client_secret=$AZURE_CLIENT_SECRET

# Create Terraform role
vault write azure/roles/terraform-deploy \
  ttl=1h \
  azure_roles=-<<EOF
  [
    {
      "role_name": "Contributor",
      "scope": "/subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/my-rg"
    }
  ]
EOF
```

**Terraform Usage:**

```hcl
data "vault_azure_access_credentials" "terraform" {
  backend = "azure"
  role    = "terraform-deploy"
  ttl     = "15m"
}

provider "azurerm" {
  features {}
  
  subscription_id = var.azure_subscription_id
  tenant_id       = var.azure_tenant_id
  client_id       = data.vault_azure_access_credentials.terraform.client_id
  client_secret   = data.vault_azure_access_credentials.terraform.client_secret
}
```

---

## 🔐 Application Secrets Management

### **Pattern: Secrets from Vault to Cloud Secret Stores**

**Common scenario**: Deploy application that needs database password.

**Anti-pattern:** Store secret in Terraform state

```hcl
# ❌ DON'T DO THIS
resource "aws_db_instance" "app" {
  password = var.db_password  # Now in state file!
}
```

**Better pattern:** Retrieve from Vault, store in cloud secret manager

```hcl
# Retrieve secret from Vault
data "vault_kv_secret_v2" "db_password" {
  mount = "secret"
  name  = "database/app-db"
}

# Store in AWS Secrets Manager
resource "aws_secretsmanager_secret" "db_password" {
  name = "app/database/password"
}

resource "aws_secretsmanager_secret_version" "db_password" {
  secret_id     = aws_secretsmanager_secret.db_password.id
  secret_string = data.vault_kv_secret_v2.db_password.data["password"]
}

# Application retrieves from Secrets Manager
resource "aws_db_instance" "app" {
  # Password not in Terraform
  manage_master_user_password = true
  master_user_secret_kms_key_id = aws_kms_key.db.id
}
```

### **Pattern: Dynamic Database Credentials**

**Vault can generate database credentials on-demand:**

**Vault Setup:**

```bash
# Enable database secrets engine
vault secrets enable database

# Configure PostgreSQL
vault write database/config/mydb \
  plugin_name=postgresql-database-plugin \
  allowed_roles="app-role" \
  connection_url="postgresql://{{username}}:{{password}}@postgres.example.com:5432/mydb" \
  username="vault-admin" \
  password="vault-admin-password"

# Create role
vault write database/roles/app-role \
  db_name=mydb \
  creation_statements="CREATE ROLE \"{{name}}\" WITH LOGIN PASSWORD '{{password}}' VALID UNTIL '{{expiration}}'; GRANT SELECT, INSERT, UPDATE ON ALL TABLES IN SCHEMA public TO \"{{name}}\";" \
  default_ttl="1h" \
  max_ttl="24h"
```

**Terraform Usage:**

```hcl
# Get dynamic DB credentials from Vault
data "vault_database_secret_backend_connection" "postgres" {
  backend       = "database"
  name          = "app-role"
  ttl           = 3600
}

# Store in AWS Secrets Manager for application
resource "aws_secretsmanager_secret_version" "app_db_creds" {
  secret_id = aws_secretsmanager_secret.app_db.id
  secret_string = jsonencode({
    username = data.vault_database_secret_backend_connection.postgres.username
    password = data.vault_database_secret_backend_connection.postgres.password
  })
}
```

---

## 🤝 Improving Vault + Terraform Team Synergy

### **Common Organizational Issues**

| Issue | Impact | Solution |
|-------|--------|----------|
| **No shared documentation** | Duplication, inconsistent patterns | Joint wiki/runbook |
| **Separate backlogs** | Misaligned priorities | Shared roadmap meetings |
| **Different on-call rotations** | Slow incident response | Cross-team escalation paths |
| **No design reviews** | Integration issues discovered late | Joint architecture reviews |
| **Blame culture** | Finger-pointing during incidents | Blameless postmortems |

### **Collaboration Improvements**

**1. Joint Design Reviews**
- Schedule: Bi-weekly
- Attendees: 2 from Vault team + 2 from Terraform team
- Topics: New secret engines, auth methods, policy changes, Terraform patterns

**2. Shared Documentation**
- **"Golden Path" Examples**: Pre-approved patterns for common scenarios
- **Troubleshooting Runbook**: Common issues and resolutions
- **Onboarding Guide**: How to use Vault from Terraform

**3. Service Level Agreements (SLAs)**
- Vault availability: 99.9% uptime
- New policy creation: < 2 business days
- Incident response: < 30 minutes (P1), < 4 hours (P2)

**4. Cross-Training**
- Vault team learns Terraform basics
- Terraform team learns Vault concepts
- Quarterly knowledge sharing sessions

---

## 🗣️ Discussion Guide: Building Alignment

### **Questions to Ask**

**1. "Would it help if Terraform and Vault teams had a joint design session?"**

*Propose specific outcomes:*
- Document 5 common Terraform + Vault patterns
- Create shared troubleshooting guide
- Establish escalation path
- Define SLAs

**2. "Can we create a 'golden path' example showing Terraform + Vault best practices?"**

*Concrete deliverable:*
- GitHub repo with working examples
- Authentication (AppRole, JWT, K8s)
- Dynamic AWS/Azure credentials
- Secret management patterns
- CI/CD integration

**3. "What blockers prevent using Vault's dynamic secret engines?"**

*Common blockers and solutions:*

| Blocker | Solution |
|---------|----------|
| "We don't know how" | Training + documentation |
| "It's complex" | Start with one use case |
| "Vault team is overloaded" | Prioritize, show business value |
| "Our apps can't rotate credentials" | Start with Terraform itself, apps later |

**4. "Would a shared runbook for Terraform-Vault troubleshooting help?"**

*Include:*
- Token expiration: Symptoms, resolution
- Policy errors: How to debug, who to contact
- Network issues: Connectivity checks
- Performance issues: When to engage Vault team

---

## 📋 Terraform + Vault Maturity Model

### **Level 0: No Integration**
- Credentials hardcoded or in environment variables
- No Vault usage
- **Risk**: Extremely high

### **Level 1: Static Secrets**
- Secrets stored in Vault
- Retrieved via `vault_generic_secret`
- Long-lived credentials
- **Risk**: High (Vault used as password manager)

### **Level 2: Dynamic Credentials**
- Cloud credentials generated by Vault
- Short TTLs (< 1 hour)
- Automatic revocation
- **Risk**: Low (significant improvement)

### **Level 3: Full Automation**
- OIDC/JWT authentication (no secrets!)
- Dynamic cloud credentials
- Dynamic database credentials
- Policy-as-code
- Full audit logging
- **Risk**: Very low (best practice)

### **Goal: Reach Level 3**

---

## 💡 Key Messages to Emphasize

1. **Vault Is Not a Password Manager** - Use dynamic secrets, not static storage.

2. **Authentication Matters** - OIDC/JWT > AppRole > Token (long-lived).

3. **Short TTLs Are Your Friend** - 15-minute credentials can't be exploited if leaked.

4. **Cross-Team Collaboration Is Essential** - Terraform and Vault succeed together or fail together.

5. **Start Simple, Iterate** - Begin with Terraform's own credentials, expand to applications later.

---

**Next Steps:**
- [Business Case & Change Management](./06-business-case-and-change.md)
- [Technical Workshops](./07-technical-workshops.md)

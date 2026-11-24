# openshift-ai-bootstrap

AgnosticD v2 deployment of Red Hat OpenShift AI (RHOAI) on OpenShift 4.22,
using Helm + GitOps (Argo CD). Supports AWS (primary), GCP, and Azure.

## Architecture

- **Infra layer**: AgnosticD v2 provisions OCP 4.22 on the target cloud provider
- **Workload layer**: `ocp4_workload_rhoai` deploys RHOAI via Argo CD + `rhai-on-openshift-chart`
- **GPU**: NFD + NVIDIA GPU Operator managed by the RHOAI Helm chart
- **Tracking**: spec-status v0.4.1 against a speckit-roadmap v0.1.0

## Quick start

```bash
# 1. Clone this repo and source repos
git clone https://github.com/tosin2013/openshift-ai-bootstrap
cd openshift-ai-bootstrap
git clone https://github.com/tosin2013/agnosticd-v2 ~/agnosticd-v2
git clone https://github.com/opendatahub-io/odh-gitops ~/odh-gitops
cd ~/odh-gitops && git checkout rhoai-devtest && cd -

# 2. Install the RHOAI workload role into the fork
cp -r roles/ocp4_workload_rhoai ~/agnosticd-v2/ansible/roles/

# 3. Set up AgnosticD
cd ~/agnosticd-v2 && ./bin/agd setup && cd -

# 4. Create secrets (never commit this file)
mkdir -p ~/agnosticd-v2-secrets
cp vars/secrets-template.yml ~/agnosticd-v2-secrets/secrets.yml
# Edit ~/agnosticd-v2-secrets/secrets.yml with your pull secret and cloud creds

# 5. Check spec roadmap progress
spec status

# 6. Deploy on AWS
agd provision \
  --vars vars/rhoai-aws.yml \
  --secrets ~/agnosticd-v2-secrets/
```

## Cloud providers

| Provider | Vars file | GPU instance |
|---|---|---|
| AWS | `vars/rhoai-aws.yml` | `g4dn.xlarge` (NVIDIA T4) |
| GCP | `vars/rhoai-gcp.yml` | `n1-standard-4` + T4 accelerator |
| Azure | `vars/rhoai-azure.yml` | `Standard_NC6s_v3` (NVIDIA V100) |

## Verification

```bash
agd status --vars vars/rhoai-aws.yml
oc get datasciencecluster -n redhat-ods-operator
oc get route rhods-dashboard -n redhat-ods-applications -o jsonpath='{.spec.host}'
```

## OCP 5 support

Tracked in `specs/006-ocp5-support/` — planned, unblocked once a stable
AgnosticD installer config for OCP 5 is confirmed.

## Related repos

- [tosin2013/agnosticd-v2](https://github.com/tosin2013/agnosticd-v2) — AgnosticD fork
- [opendatahub-io/odh-gitops](https://github.com/opendatahub-io/odh-gitops) — RHOAI GitOps charts (branch: `rhoai-devtest`)
- [tosin2013/rhel-devops-skills-cli](https://github.com/tosin2013/rhel-devops-skills-cli) — Installer used to scaffold this project

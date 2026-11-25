# Plan: AWS Deployment (OCP 4.22 + RHOAI + GPU)

## Implementation Approach

This spec is a pure operational execution — no new code is authored. The
implementation consumes the already-verified artifacts from specs 001 and 002.

## Stack

- **Cluster provisioner**: AgnosticD `openshift-cluster` config (tosin2013/agnosticd-v2)
- **Cluster version**: OCP 4.22, AWS `us-east-2`
- **Worker nodes**: 3 × `m6i.4xlarge`
- **GPU pool**: 1 × `g4dn.xlarge` (NVIDIA T4, via `ocp4_workload_machinesets`)
- **RHOAI installation**: `ocp4_workload_rhoai` role → Argo CD → `rhai-on-openshift-chart` v3.4
- **GitOps source**: `opendatahub-io/odh-gitops` branch `rhoai-devtest`

## Execution Sequence

```
Phase 1 — Credentials & Setup
  aws configure                       ← AWS CLI auth (us-east-2)
  agd setup                           ← creates virtualenv
  Fill secrets-aws.yml                ← pull_secret + registry creds

Phase 2 — Pre-flight
  check-quota.sh aws us-east-2        ← verify EC2 quota for g4dn.xlarge

Phase 3 — Provision
  agd provision \
    --guid <GUID> \
    --config openshift-cluster \
    --account <ACCOUNT>               ← provisions cluster + runs workloads

Phase 4 — Verify
  oc get datasciencecluster           ← RHOAI platform ready
  oc get route rhods-dashboard        ← dashboard accessible
  oc get nodes -l nvidia.com/gpu=true ← GPU node allocatable

Phase 5 — Teardown test
  agd destroy \
    --guid <GUID> \
    --config openshift-cluster \
    --account <ACCOUNT>               ← confirm clean teardown
```

## Workloads Applied (in order)

1. `ocp4_workload_cert_manager` — TLS certificates
2. `ocp4_workload_machinesets` — GPU MachineSet (g4dn.xlarge)
3. `ocp4_workload_openshift_gitops` — Argo CD
4. `ocp4_workload_rhoai` — RHOAI via Argo CD + Helm chart

## Vars File

`vars/rhoai-aws.yml` — already present, contains all required configuration.

## Secrets Required

`~/agnosticd-v2-secrets/secrets-aws.yml`:
- `ocp4_pull_secret` — from console.redhat.com
- `ocp4_workload_rhoai_registry_username` / `_password` — registry service account
- AWS credentials — from `secrets-<account>.yml`

## Risk

- Q1 (OLM channel) — if chart deploys with wrong channel, RHOAI operator install
  will hang. Resolve by checking `oc get subscription rhods-operator` channel.
- GPU node quota — g4dn.xlarge requires EC2 quota in us-east-2. Run check-quota.sh first.

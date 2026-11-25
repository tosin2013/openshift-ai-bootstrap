# Plan: Azure Deployment (OCP 4.22 + RHOAI + GPU)

## Implementation Approach

Same operational pattern as specs 003 and 004. Azure-specific vars only.

## Stack

- **Cluster provisioner**: AgnosticD `openshift-cluster` (tosin2013/agnosticd-v2)
- **Cluster version**: OCP 4.22, Azure `eastus`
- **GPU pool**: 1 × `Standard_NC6s_v3` (NVIDIA V100, via `ocp4_workload_machinesets`)
- **RHOAI**: `ocp4_workload_rhoai` → Argo CD → `rhai-on-openshift-chart` v3.4

## Execution Sequence

```
Phase 1 — Credentials & Setup
  az login && az account set --subscription <SUBSCRIPTION_ID>
  agd setup (if not already done)
  Fill secrets-azure.yml

Phase 2 — Pre-flight
  check-quota.sh azure eastus

Phase 3 — Provision
  agd provision --guid <GUID> --config openshift-cluster --account <ACCOUNT>

Phase 4 — Verify
  oc get datasciencecluster / route rhods-dashboard / nodes -l nvidia.com/gpu=true

Phase 5 — Teardown test
  agd destroy --guid <GUID> --config openshift-cluster --account <ACCOUNT>
```

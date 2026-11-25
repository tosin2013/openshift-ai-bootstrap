# Plan: GCP Deployment (OCP 4.22 + RHOAI + GPU)

## Implementation Approach

Same operational pattern as spec 003 (AWS). No new code — consumes spec 001/002
artifacts with GCP-specific vars.

## Stack

- **Cluster provisioner**: AgnosticD `openshift-cluster` (tosin2013/agnosticd-v2)
- **Cluster version**: OCP 4.22, GCP `us-east1`
- **GPU pool**: 1 × `n1-standard-4` + NVIDIA T4 accelerator (via `ocp4_workload_machinesets`)
- **RHOAI**: `ocp4_workload_rhoai` → Argo CD → `rhai-on-openshift-chart` v3.4

## Execution Sequence

```
Phase 1 — Credentials & Setup
  gcloud auth login && gcloud config set project <PROJECT_ID>
  agd setup (if not already done)
  Fill secrets-gcp.yml

Phase 2 — Pre-flight
  check-quota.sh gcp us-east1

Phase 3 — Provision
  agd provision --guid <GUID> --config openshift-cluster --account <ACCOUNT>

Phase 4 — Verify
  oc get datasciencecluster / route rhods-dashboard / nodes -l nvidia.com/gpu=true

Phase 5 — Teardown test
  agd destroy --guid <GUID> --config openshift-cluster --account <ACCOUNT>
```

## Risk

- Q2 (GCP accelerator field) — if `ocp4_workload_machinesets` does not support
  the `accelerator` field natively, a custom MachineSet YAML will be needed.
  Verify against role implementation in `~/agnosticd-v2/ansible/roles/ocp4_workload_machinesets/`.

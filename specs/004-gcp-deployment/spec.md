# Spec: GCP Deployment (OCP 4.22 + RHOAI + GPU)

## Overview

Provision OCP 4.22 on GCP and install RHOAI using the same `ocp4_workload_rhoai`
role used for AWS. This is the second cloud target, confirming the role works
cloud-agnostically. GPU pool uses `g2-standard-4` (NVIDIA L4, 24GB VRAM).

**Key GCP differences from AWS:**
- Cloud provider: `cloud_provider_repo: https://github.com/tosin2013/cloud_provider_gcp.git`
- Credentials: GCP service account JSON key file (`gcp_credentials_file`)
- GPU: `g2-standard-4` (purpose-built L4 machine type, no guest accelerator attachment)
- MachineSet: `ocp4_workload_machinesets` has no GCP implementation — GPU MachineSet created directly by `ocp4_workload_rhoai` via `kubernetes.core.k8s`

## Functional Requirements

- FR-01: Provision OCP 4.22 on GCP (`us-east1`) using `openshift-cluster` deployer with `cloud_provider_repo` pointing at tosin2013/cloud_provider_gcp.
- FR-02: Create a GPU MachineSet (`g2-standard-4`, 1 replica, `nvidia.com/gpu` taint) via a custom `kubernetes.core.k8s` task (not `ocp4_workload_machinesets` — no GCP implementation exists).
- FR-03: Install RHOAI (`default` profile) via `ocp4_workload_rhoai`.
- FR-04: RHOAI dashboard accessible at `rhods-dashboard` route.
- FR-05: GPU node reports `nvidia.com/gpu` as allocatable.
- FR-06: `agnosticd_user_info` emits dashboard URL and profile.
- FR-07: Teardown completes cleanly (Cloud DNS records removed by cloud provider `dns` role automatically — no manual cleanup needed unlike AWS Route53).
- FR-08: GCP service account JSON credentials file configured before provision.

## Success Criteria

- SC-01: `oc get datasciencecluster -n redhat-ods-operator` shows Ready.
- SC-02: `oc get route rhods-dashboard -n redhat-ods-applications` returns a host.
- SC-03: `oc get nodes -l nvidia.com/gpu=true` returns at least one GPU node with `nvidia.com/gpu` allocatable.
- SC-04: `agd destroy` exits 0 with no GCP resources remaining.

## Constraints

- OCP 4.22 only (C-01)
- RHOAI `default` profile (C-02)
- GPU: `g2-standard-4` — NVIDIA L4, 24GB VRAM (C-03; T4 deprecated Aug 2027)
- GitOps via Argo CD (C-04)
- Cloud provider: tosin2013/cloud_provider_gcp fork

## Open Questions

- Q2: **RESOLVED** — `ocp4_workload_machinesets` has no GCP implementation (`machineset-gcp.yml` does not exist). Solution: create GCP MachineSet CR directly in `ocp4_workload_rhoai` workload.yml using `kubernetes.core.k8s`, extracting cluster facts (project, network, zone, CoreOS image) from existing worker MachineSets.

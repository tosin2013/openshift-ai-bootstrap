# Spec: GCP Deployment (OCP 4.22 + RHOAI + GPU)

## Overview

Provision OCP 4.22 on GCP and install RHOAI using the same `ocp4_workload_rhoai`
role used for AWS. This is the second cloud target, confirming the role works
cloud-agnostically. GPU pool uses `n1-standard-4` with an attached NVIDIA T4
accelerator.

## Functional Requirements

- FR-01: Provision OCP 4.22 on GCP (`us-east1`) using `openshift-cluster` deployer.
- FR-02: Add a GPU-capable MachineSet (`n1-standard-4` + NVIDIA T4 accelerator).
- FR-03: Install RHOAI (`default` profile) via `ocp4_workload_rhoai`.
- FR-04: RHOAI dashboard accessible at `rhods-dashboard` route.
- FR-05: GPU node reports `nvidia.com/gpu` as allocatable.
- FR-06: `agnosticd_user_info` emits dashboard URL and profile.
- FR-07: Teardown completes cleanly.

## Success Criteria

- SC-01: `oc get datasciencecluster -n redhat-ods-operator` shows Ready.
- SC-02: `oc get route rhods-dashboard -n redhat-ods-applications` returns a host.
- SC-03: `oc get nodes -l nvidia.com/gpu=true` returns at least one GPU node.
- SC-04: `agd destroy` exits 0 with no GCP resources remaining.

## Open Questions

- Q2: Does `ocp4_workload_machinesets` support the GCP `accelerator` field, or
  does it require a custom MachineSet manifest? Resolve during this spec.

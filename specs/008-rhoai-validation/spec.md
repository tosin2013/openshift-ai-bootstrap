# Spec: RHOAI Post-Deploy Validation

## Overview

A cross-cloud validation role (`ocp4_workload_validate_rhoai`) that runs as the
final workload after every RHOAI deployment (AWS, GCP, Azure) to confirm the full
GPU + AI stack is healthy before the deployment is marked `verified`. This is the
release gate: a deployment cannot be tagged as a release until all checks pass.

## Functional Requirements

- FR-01: Validate `DataScienceCluster` is in `Ready` phase.
- FR-02: Validate the RHOAI dashboard URL returns HTTP 200/302 (not just route exists).
- FR-03: Validate the NFD `NodeFeatureDiscovery` instance is in `Ready` condition.
- FR-04: Validate the GPU node has the NFD PCI label `feature.node.kubernetes.io/pci-10de.present=true` (confirms NVIDIA T4/A10/V100 detected).
- FR-05: Validate the GPU Operator `ClusterPolicy` is in `Ready` state.
- FR-06: Validate `nvidia.com/gpu` is allocatable (> 0) on the GPU node.
- FR-07: Validate GPU Operator daemonset pods (driver, device-plugin, toolkit) are all `Running`.
- FR-08: Validate RHOAI application pods in `redhat-ods-applications` are all `Running`.
- FR-09: Emit a structured pass/fail summary via `agnosticd.core.agnosticd_user_info`.
- FR-10: Fail the workload (non-zero exit) if ANY required check fails, blocking `verified`.

## Success Criteria

- SC-01: All 8 checks pass on a live deployment → `agnosticd_user_info` emits `VALIDATION: PASS`.
- SC-02: Any failed check → `agnosticd_user_info` emits the failing check name and reason → workload exits non-zero.
- SC-03: Role is cloud-agnostic — identical checks work on AWS, GCP, and Azure deployments.
- SC-04: Role runs idempotently — safe to re-run without side effects.
- SC-05: Total runtime under 5 minutes (pure read-only checks, no installation).

## Constraints

- Read-only: no resources are created or modified
- Cloud-agnostic: no cloud-provider-specific logic
- Works against any cluster with RHOAI + NFD + GPU Operator installed
- Must be listed as the last entry in `workloads:` after `ocp4_workload_rhoai`

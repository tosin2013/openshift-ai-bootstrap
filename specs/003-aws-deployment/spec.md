# Spec: AWS Deployment (OCP 4.22 + RHOAI + GPU)

## Overview

Provision a production-representative OCP 4.22 cluster on AWS and install Red Hat
OpenShift AI (RHOAI) using the `ocp4_workload_rhoai` Ansible role. This is the
primary cloud target and the first deployment that proves the RHOAI workload role
works end-to-end before GCP and Azure are attempted.

## Functional Requirements

- FR-01: Provision an OCP 4.22 cluster on AWS (`us-east-2`) using the
  `openshift-cluster` AgnosticD deployer with 3 `m6i.4xlarge` worker nodes.
- FR-02: Add a GPU-capable MachineSet (`g4dn.xlarge`, 1 replica) with the
  `nvidia.com/gpu: "true"` label and NoSchedule taint.
- FR-03: Install RHOAI using the `default` profile via the `ocp4_workload_rhoai`
  role. NFD and NVIDIA GPU Operator are managed by the RHOAI Helm chart.
- FR-04: The RHOAI dashboard must be accessible at the `rhods-dashboard` route in
  the `redhat-ods-applications` namespace.
- FR-05: The GPU node must report `nvidia.com/gpu` as an allocatable resource.
- FR-06: `agnosticd_user_info` must emit the dashboard URL and profile.
- FR-07: Teardown (`agd destroy`) must complete cleanly with no orphaned resources.

## Success Criteria

- SC-01: `oc get datasciencecluster -n redhat-ods-operator` shows Ready condition.
- SC-02: `oc get route rhods-dashboard -n redhat-ods-applications` returns a host.
- SC-03: `oc get nodes -l nvidia.com/gpu=true` returns at least one node with
  `nvidia.com/gpu` allocatable.
- SC-04: `agd destroy` exits 0 with no AWS resources remaining (verify via console
  or `aws ec2 describe-instances`).

## Constraints

- OCP 4.22 only (C-01)
- RHOAI `default` profile (C-02)
- GPU nodes from day 1 — g4dn.xlarge (C-03)
- GitOps via Argo CD — not direct helm install (C-04)

## Open Questions

- Q1: Does rhai-on-openshift-chart v3.4 require channel `stable` instead of
  `beta` on OCP 4.22? Resolve by checking the installed OLM Subscription channel.

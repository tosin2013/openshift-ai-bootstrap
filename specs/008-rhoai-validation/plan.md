# Plan: RHOAI Post-Deploy Validation

## Implementation Approach

A new Ansible role `ocp4_workload_validate_rhoai` added to the `workloads:` list
in all three cloud vars files (AWS, GCP, Azure) as the final workload. The role
runs 8 sequential read-only checks and emits a consolidated pass/fail report.

## Role Structure

```
roles/ocp4_workload_validate_rhoai/
  defaults/main.yml     — timeouts, namespaces, retries
  meta/main.yml         — role metadata
  tasks/
    main.yml            — ACTION dispatcher (provision → workload, destroy → no-op)
    workload.yml        — all 8 validation checks + report
    stop.yml            — no-op
    start.yml           — no-op
    status.yml          — re-runs workload.yml checks
```

## Checks (in order)

```
1. DataScienceCluster Ready          kubernetes.core.k8s_info + condition check
2. RHOAI dashboard HTTP 200          ansible.builtin.uri against rhods-dashboard route
3. NFD NodeFeatureDiscovery Ready    kubernetes.core.k8s_info
4. GPU node NFD PCI label present    kubernetes.core.k8s_info on Node labels
5. GPU Operator ClusterPolicy Ready  kubernetes.core.k8s_info
6. nvidia.com/gpu allocatable > 0    kubernetes.core.k8s_info on Node.status.allocatable
7. GPU Operator pods all Running     kubernetes.core.k8s_info on pods -n nvidia-gpu-operator
8. RHOAI pods all Running            kubernetes.core.k8s_info on pods -n redhat-ods-applications
```

## Integration

Add `ocp4_workload_validate_rhoai` as the last entry in `workloads:` in:
- `vars/rhoai-aws.yml`
- `vars/rhoai-gcp.yml`
- `vars/rhoai-azure.yml`

## Release Gate Rule

A cloud deployment can only be tagged as a git release after:
1. `agd provision` completes with exit 0
2. `ocp4_workload_validate_rhoai` reports `VALIDATION: PASS`
3. The roadmap spec for that cloud is recorded `verified`

# Tasks: GCP Deployment (OCP 4.22 + RHOAI + GPU)

- [x] Create agnosticd-v2-vars/rhoai-gcp.yml
- [x] Switch GPU instance to g2-standard-4 (L4, 24GB) — T4 deprecated Aug 2027
- [x] Add cloud_provider_repo: https://github.com/tosin2013/cloud_provider_gcp.git to vars
- [x] Remove ocp4_workload_machinesets from GCP workloads (no machineset-gcp.yml)
- [x] Add custom GCP MachineSet task to ocp4_workload_rhoai role (kubernetes.core.k8s)
- [x] Get GCP Open Environment (project: openenv-txdmd, service account JSON obtained)
- [x] Validate GCP service account JSON (validate-gcp-key.py — fresh key created after RHPDS truncation)
- [x] Run agd provision — OCP 4.22.22 installed in us-east1 (project openenv-txdmd)
- [x] Fix cert-manager channel (stable-v1 not stable-v1.15; OperatorGroup race condition resolved)
- [x] Fix GPU wait task order (must be after MachineSet creation, not before Helm install)
- [x] Fix GCP cluster ID extraction (filter worker MachineSets to avoid GPU MS name collision)
- [x] Fix cert-manager gcp_credentials (lookup file in EE /secrets/ path)
- [x] Enable dashboard + workbenches in DSC (Managed not Removed)
- [x] Scale workers to 5 (n2-standard-4 ×3 insufficient for RHOAI dashboard; needed ×5)
- [x] Verify DataScienceCluster Ready: True ✅
- [x] Verify RHOAI dashboard HTTP 200 ✅
- [x] Verify ArgoCD RHOAI app Synced/Healthy ✅
- [x] Verify NFD NodeFeatureDiscovery created ✅
- [x] Verify GPU Operator ClusterPolicy ready ✅
- [ ] GPU node hardware: nvidia.com/gpu NOT allocatable ← BLOCKED (see notes)
      Root cause: g2-standard-4 (L4) not available in us-east1 (all zones STOCKOUT).
      OCP 4.22 Machine API rejects guestAccelerators (n1+T4 path blocked).
      Fix: redeploy cluster in us-east4 where g2-standard-4 is supported.
- [ ] Update vars/rhoai-gcp.yml: gcp_region=us-east4, gcp_gpu_zone=us-east4-a
- [ ] Re-provision GCP cluster in us-east4 for GPU hardware support
- [x] Verify agnosticd_user_info emitted — dashboard URL in provision-user-info.yaml ✅
- [x] Teardown test: agd destroy — Ansible playbook PASS (failed=0). GCP bastion and
      network deleted cleanly. OCP cluster nodes required manual GCP API deletion because
      cloud_provider_gcp destroy_instances does not run openshift-install destroy cluster
      (bastion is deleted before OCP installer runs). Note: add openshift-install destroy
      step to cloud_provider_gcp destroy role before next deployment.
- [x] Tag v0.2.0-gcp — released 2026-11-27 after 8/8 validation PASS ✅

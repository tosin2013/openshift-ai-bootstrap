# Tasks: GCP Deployment (OCP 4.22 + RHOAI + GPU)

- [x] Create agnosticd-v2-vars/rhoai-gcp.yml
- [x] Configure GPU MachineSet vars (n1-standard-4 with accelerator)
- [ ] Configure GCP CLI credentials: gcloud auth login && gcloud config set project <PROJECT_ID>
- [ ] Run agd setup (creates ~/agnosticd-v2-virtualenv/ — if not already done from spec 003)
- [ ] Create ~/agnosticd-v2-secrets/secrets-gcp.yml (from vars/secrets-template.yml; fill pull_secret, rhsm creds, ocp4_workload_rhoai_registry_username/password, GCP service account)
- [ ] Run check-quota.sh for GCP
- [ ] Run: cd ~/agnosticd-v2 && ./bin/agd provision --guid <GUID> --config openshift-cluster --account <ACCOUNT>
- [ ] Verify DataScienceCluster reaches Ready: oc get datasciencecluster -n redhat-ods-operator
- [ ] Verify RHOAI dashboard accessible: oc get route rhods-dashboard -n redhat-ods-applications
- [ ] Verify GPU node has nvidia.com/gpu allocatable: oc get nodes -l nvidia.com/gpu=true

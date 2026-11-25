# Tasks: Azure Deployment (OCP 4.22 + RHOAI + GPU)

- [x] Create agnosticd-v2-vars/rhoai-azure.yml
- [x] Configure GPU MachineSet vars (Standard_NC6s_v3)
- [ ] Configure Azure CLI credentials: az login && az account set --subscription <SUBSCRIPTION_ID>
- [ ] Run agd setup (creates ~/agnosticd-v2-virtualenv/ — if not already done from spec 003)
- [ ] Create ~/agnosticd-v2-secrets/secrets-azure.yml (from vars/secrets-template.yml; fill pull_secret, rhsm creds, ocp4_workload_rhoai_registry_username/password, Azure subscription/tenant/client creds)
- [ ] Run check-quota.sh for Azure
- [ ] Run: cd ~/agnosticd-v2 && ./bin/agd provision --guid <GUID> --config openshift-cluster --account <ACCOUNT>
- [ ] Verify DataScienceCluster reaches Ready: oc get datasciencecluster -n redhat-ods-operator
- [ ] Verify RHOAI dashboard accessible: oc get route rhods-dashboard -n redhat-ods-applications
- [ ] Verify GPU node has nvidia.com/gpu allocatable: oc get nodes -l nvidia.com/gpu=true

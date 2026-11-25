# Tasks: AWS Deployment (OCP 4.22 + RHOAI + GPU)

- [x] Create agnosticd-v2-vars/rhoai-aws.yml with openshift-cluster deployer
- [x] Configure GPU MachineSet vars (g4dn.xlarge, 1 replica)
- [ ] Configure AWS CLI credentials: aws configure (access key, secret, region us-east-2, output json)
- [ ] Run agd setup (creates ~/agnosticd-v2-virtualenv/ — required before provision)
- [ ] Create ~/agnosticd-v2-secrets/secrets-aws.yml (from vars/secrets-template.yml; fill pull_secret, rhsm creds, ocp4_workload_rhoai_registry_username/password)
- [ ] Run check-quota.sh for AWS
- [ ] Run: cd ~/agnosticd-v2 && ./bin/agd provision --guid <GUID> --config openshift-cluster --account <ACCOUNT>
- [ ] Verify DataScienceCluster reaches Ready: oc get datasciencecluster -n redhat-ods-operator
- [ ] Verify RHOAI dashboard accessible: oc get route rhods-dashboard -n redhat-ods-applications
- [ ] Verify GPU node has nvidia.com/gpu allocatable: oc get nodes -l nvidia.com/gpu=true

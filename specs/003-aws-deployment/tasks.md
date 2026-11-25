# Tasks: AWS Deployment (OCP 4.22 + RHOAI + GPU)

- [x] Create agnosticd-v2-vars/rhoai-aws.yml with openshift-cluster deployer
- [x] Configure GPU MachineSet vars (g4dn.xlarge, 1 replica)
- [x] Configure AWS CLI credentials: aws configure (access key, secret, region us-east-2, output json)
- [x] Run agd setup (creates ~/agnosticd-v2-virtualenv/ — required before provision)
- [ ] Fill ~/agnosticd-v2-secrets/secrets.yml: set ocp4_pull_secret (console.redhat.com) and ocp4_workload_rhoai_registry_username/password (access.redhat.com/terms-based-registry/)
- [ ] Confirm ~/agnosticd-v2-secrets/secrets-zppkh.yml has base_domain: sandbox1044.opentlc.com and AWS credentials set
- [ ] Run check-quota.sh for AWS
- [ ] Run: cd ~/agnosticd-v2 && ./bin/agd provision --guid rhoai01 --config openshift-cluster --account zppkh
- [ ] Verify DataScienceCluster reaches Ready: oc get datasciencecluster -n redhat-ods-operator
- [ ] Verify RHOAI dashboard accessible: oc get route rhods-dashboard -n redhat-ods-applications
- [ ] Verify GPU node has nvidia.com/gpu allocatable: oc get nodes -l nvidia.com/gpu=true
- [ ] Verify agnosticd_user_info emitted: check agnosticd-v2-output/rhoai01/ for user_info output with dashboard URL
- [ ] Teardown test: cd ~/agnosticd-v2 && ./bin/agd destroy --guid rhoai01 --config openshift-cluster --account zppkh

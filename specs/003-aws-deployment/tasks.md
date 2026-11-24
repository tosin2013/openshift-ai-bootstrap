# Tasks: AWS Deployment (OCP 4.22 + RHOAI + GPU)

- [ ] Create agnosticd-v2-vars/rhoai-aws.yml with openshift-cluster deployer
- [ ] Configure GPU MachineSet vars (g4dn.xlarge, 1 replica)
- [ ] Create agnosticd-v2-secrets/secrets-aws.yml (from template, fill in creds)
- [ ] Run check-quota.sh for AWS
- [ ] Run agd provision --config rhoai-aws
- [ ] Verify DataScienceCluster reaches Ready
- [ ] Verify RHOAI dashboard accessible
- [ ] Verify GPU node has nvidia.com/gpu allocatable

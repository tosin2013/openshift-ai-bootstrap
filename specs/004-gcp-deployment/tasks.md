# Tasks: GCP Deployment (OCP 4.22 + RHOAI + GPU)

- [ ] Create agnosticd-v2-vars/rhoai-gcp.yml
- [ ] Configure GPU MachineSet vars (n1-standard-4 with accelerator)
- [ ] Create agnosticd-v2-secrets/secrets-gcp.yml
- [ ] Run check-quota.sh for GCP
- [ ] Run agd provision --config rhoai-gcp
- [ ] Verify DataScienceCluster reaches Ready
- [ ] Verify RHOAI dashboard accessible

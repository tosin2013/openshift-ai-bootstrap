# Tasks: ocp4_workload_rhoai Ansible Role

- [ ] Create role directory structure in agnosticd-v2/ansible/roles/ocp4_workload_rhoai/
- [ ] Write defaults/main.yml with chart_version, operator_type, profile, gpu_enabled vars
- [ ] Write tasks/main.yml: install Helm v4, create pull secret namespace
- [ ] Write tasks/provision.yml: create Argo CD Application CR for rhai-on-openshift-chart
- [ ] Write tasks/wait.yml: wait loop for DataScienceCluster Ready condition
- [ ] Write tasks/user_info.yml: emit agnosticd_user_info with dashboard URL
- [ ] Write playbooks: provision.yml, destroy.yml, stop.yml, start.yml, status.yml
- [ ] Test role syntax check (ansible-lint)

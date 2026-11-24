# Tasks: ocp4_workload_rhoai Ansible Role

- [x] Create role directory structure in agnosticd-v2/ansible/roles/ocp4_workload_rhoai/
- [x] Write defaults/main.yml with chart_version, operator_type, profile, gpu_enabled vars
- [x] Write tasks/main.yml: ACTION dispatcher (provision/destroy)
- [x] Write tasks/workload.yml: install Helm v4, create pull secret, deploy Argo CD Application CR
- [x] Write tasks/remove.yml: remove Argo CD Application and namespaces
- [x] Write tasks/wait.yml equivalent: wait for DataScienceCluster Ready + emit agnosticd_user_info
- [x] Copy role into bootstrap repo under roles/ocp4_workload_rhoai/
- [x] Test role syntax check (ansible-lint) against agnosticd-v2 EE image

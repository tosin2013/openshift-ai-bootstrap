# Tasks: CI Version-Drift Detection

- [ ] Create .github/workflows/version-check.yml (scheduled weekly)
- [ ] Add OCP release check: query quay.io/openshift-release-dev/ocp-release tags; compare against vars/rhoai-aws.yml host_ocp4_installer_version
- [ ] Add RHOAI chart check: query registry.redhat.io/rhai OCI tags; compare against roles/ocp4_workload_rhoai/defaults/main.yml ocp4_workload_rhoai_chart_version
- [ ] Add odh-gitops branch check: GitHub API last commit SHA on rhoai-devtest; compare against pinned gitops_branch default
- [ ] Configure GitHub issue creation on drift (gh issue create with diff table)
- [ ] Test workflow manually with workflow_dispatch trigger
- [ ] Verify issue is opened correctly on simulated drift

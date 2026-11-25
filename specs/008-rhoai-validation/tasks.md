# Tasks: RHOAI Post-Deploy Validation

- [ ] Create roles/ocp4_workload_validate_rhoai/ role structure (defaults, meta, tasks/)
- [ ] Write tasks/workload.yml with all 8 validation checks
- [ ] Write tasks/main.yml ACTION dispatcher (provision/destroy/stop/start/status)
- [ ] Write tasks/status.yml that re-runs the same checks
- [ ] Add ocp4_workload_validate_rhoai as last workload in vars/rhoai-aws.yml
- [ ] Add ocp4_workload_validate_rhoai as last workload in vars/rhoai-gcp.yml
- [ ] Add ocp4_workload_validate_rhoai as last workload in vars/rhoai-azure.yml
- [ ] Run ansible-lint on the new role
- [ ] Test validation role against live AWS deployment (spec 003)
- [ ] Confirm VALIDATION: PASS emitted in agnosticd_user_info output
- [ ] Confirm role exit non-zero when a check fails (inject a failure, verify detection)
- [ ] Add FR-09 and FR-10 to specs/003-aws-deployment/spec.md
- [ ] Tag v0.1.0 release after AWS deployment passes all validation checks
EOF

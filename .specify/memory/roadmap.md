<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0
Bump rationale: MINOR — initial formal issue; promoted from hand-written
  minimal format to full template structure. All six entries enriched with
  description, outcome, scope, notes, and constraints. No entry removed
  or restated.

Changes this revision:
  - Promoted hand-written roadmap to template-compliant structure
  - Added Vision & End States
  - Added Constraints & Decisions C-01 through C-06
  - Enriched all six Planned Specs entries with required fields
  - Added Open Questions Q1 and Q2
  - Added Cross-Cutting Notes

Specs affected: 001, 002, 003, 004, 005, 006
Open questions added/resolved: Q1 and Q2 added

Notes: Roadmap previously hand-authored as a minimal spec-status-compatible
  file. This revision adds governance context from the session in which the
  project was created (2026-11-24).
-->

# openshift-ai-bootstrap — Spec Roadmap

Living, non-binding map of the specs planned for openshift-ai-bootstrap. It is
**not a commitment to order or scope** — it captures spec-specific discussion,
decisions, technology choices, outcomes, and constraints so they are not lost
before the spec that needs them is written. Specs are scoped and clarified when
they are actually started. Foundations: session context from 2026-11-24.

Status legend (lifecycle): **undecided** · **needs-info** · **planned** ·
**specced** · **in-progress** · **implemented** · **verified** · **deferred** ·
**abandoned**.

---

## Vision & End States

- Engineers and Red Hat field teams can provision a fully operational RHOAI
  environment on OCP 4.22 — with GPU support — on AWS, GCP, or Azure by running
  a single `agd provision` command against a cloud-provider vars file.
- Each cloud-provider deployment is a discrete, verifiable spec tracked by
  speckit-roadmap: a spec unblocks only after its upstream role is recorded
  verified.
- OCP 5 support is planned and unblocks when AgnosticD confirms stable installer
  support for OCP 5.

## Constraints & Decisions

- **C-01 — OCP 4.22 as initial target:** OCP 5 support deferred until a stable
  AgnosticD installer config for OCP 5 is confirmed. The RHOAI Helm chart
  requires OCP >= 4.19.9; 4.22 is the safest confirmed version.
- **C-02 — RHOAI profile = default:** The `rhaii` profile (distributed inference,
  llm-d, Red Hat Connectivity Link, LeaderWorkerSet) is explicitly excluded from
  initial scope. It adds significant complexity and was not requested.
- **C-03 — GPU nodes from day 1:** g4dn.xlarge (AWS, 1× NVIDIA T4),
  n1-standard-4+T4 (GCP), Standard_NC6s_v3 (Azure, 1× NVIDIA V100). NFD and
  NVIDIA GPU Operator are managed by the RHOAI Helm chart (`nfd: true`,
  `nvidiaGPUOperator: true`); no separate Ansible roles required.
- **C-04 — GitOps via Argo CD:** RHOAI chart is deployed as an Argo CD
  Application (not `helm install`). Required for two-phase CRD bootstrapping
  (`SkipDryRunOnMissingResource=true`, `skipCrdCheck: true`).
- **C-05 — odh-gitops `rhoai-devtest` branch:** The `rhoai-3.4` branch
  referenced in the Aug 2026 article does not exist in the upstream
  opendatahub-io/odh-gitops repo. `rhoai-devtest` is the nearest equivalent
  and contains `prepare-authorino-tls.sh` and the `rhai-on-openshift-chart`
  values.yaml.
- **C-06 — spec-status v0.4.1 + speckit-roadmap v0.1.1:** These tools are the
  lifecycle tracking standard for this project; all status transitions go through
  `/speckit.roadmap.write`.

## Planned Specs

### 001 — AgnosticD Infra Project Scaffold  [status: implemented]

- **Description:** Clone and configure the agnosticd-v2 fork and odh-gitops
  repo; scaffold the openshift-ai-bootstrap project with onboard.yml,
  deploy/teardown/check-quota scripts, cloud-specific vars files, and a
  spec-status roadmap.
- **Outcome:** A runnable AgnosticD infra project where any engineer, after
  cloning this repo and creating a secrets file, can follow onboard.yml to
  provision RHOAI on a chosen cloud provider. Cloud vars files for AWS, GCP, and
  Azure are present. GitHub repo is public and pushed.
- **Scope (in):** agnosticd-v2 fork clone, odh-gitops clone (rhoai-devtest
  branch), scaffold generation (Makefile, bootstrap.sh, scripts/), vars files,
  secrets-template.yml, speckit-roadmap + spec-status setup, GitHub repo
  creation.
- **Scope (out):** Actual cloud provisioning; authoring the RHOAI workload role;
  cloud credentials.
- **Depends on:** none
- **Governed by:** C-06
- **Spec dir:** specs/001-agnosticd-infra-project

### 002 — ocp4_workload_rhoai Ansible Role  [status: implemented]

- **Description:** Author the ocp4_workload_rhoai Ansible role in
  agnosticd-v2/ansible/roles/. The role installs RHOAI on an existing OCP
  cluster by installing Helm v4, creating the redhat-ods-operator namespace and
  pull secret, deploying Argo CD (via ocp4_workload_openshift_gitops), and
  creating an Argo CD Application pointing at rhai-on-openshift-chart v3.4 with
  NFD and NVIDIA GPU Operator enabled.
- **Outcome:** A self-contained Ansible role that, when added to a `workloads:`
  list in an AgnosticD vars file, installs RHOAI on any OCP 4.22 cluster (AWS,
  GCP, or Azure) using the default profile with GPU support, waits for
  DataScienceCluster Ready, and emits agnosticd_user_info with the dashboard URL.
- **Scope (in):** role defaults, tasks/main.yml (ACTION dispatcher),
  tasks/workload.yml (provision), tasks/remove.yml (destroy), DataScienceCluster
  readiness wait loop, agnosticd_user_info emission.
- **Scope (out):** rhaii profile; per-cloud Helm values overrides beyond gpu/nfd
  flags; actual cloud test deployment.
- **Depends on:** 001
- **Governed by:** C-02, C-03, C-04, C-05
- **Spec dir:** specs/002-ocp4-workload-rhoai-role
- **Notes:** One task remains unchecked: ansible-lint against the agnosticd EE
  image (`specs/002-ocp4-workload-rhoai-role/tasks.md:10`). The
  `gitops_branch` default is `rhoai-devtest` per C-05.

### 003 — AWS Deployment (OCP 4.22 + RHOAI + GPU)  [status: planned]

- **Description:** Provision an OCP 4.22 cluster on AWS using the
  `openshift-cluster` AgnosticD deployer, then apply ocp4_workload_rhoai to
  install RHOAI with a g4dn.xlarge GPU node pool.
- **Outcome:** A live RHOAI dashboard accessible at the RHODS route, with one
  GPU-capable node (NVIDIA T4) visible to the RHOAI workbench. agnosticd_user_info
  emitted with dashboard URL and credentials. Deployment teardown confirmed clean.
- **Scope (in):** vars/rhoai-aws.yml provision, GPU MachineSet (g4dn.xlarge,
  1 replica), cert-manager, openshift-gitops, ocp4_workload_rhoai,
  verify-readiness check.
- **Scope (out):** GCP and Azure deployments; persistent storage (ODF); RHOAI
  model serving configuration; student workshop content.
- **Depends on:** 002
- **Governed by:** C-01, C-02, C-03, C-04
- **Spec dir:** specs/003-aws-deployment

### 004 — GCP Deployment (OCP 4.22 + RHOAI + GPU)  [status: planned]

- **Description:** Provision OCP 4.22 on GCP using the `openshift-cluster`
  AgnosticD deployer and apply ocp4_workload_rhoai. GPU node pool uses
  n1-standard-4 instances with an attached NVIDIA T4 accelerator.
- **Outcome:** Live RHOAI dashboard on GCP cluster with one GPU node.
  agnosticd_user_info emitted. Deployment teardown confirmed clean.
- **Scope (in):** vars/rhoai-gcp.yml provision, GPU MachineSet
  (n1-standard-4 + T4 accelerator).
- **Scope (out):** AWS and Azure deployments.
- **Depends on:** 002
- **Governed by:** C-01, C-02, C-03, C-04
- **Spec dir:** specs/004-gcp-deployment
- **Notes:** GCP GPU accelerator configuration uses an `accelerator` field
  rather than instance type — may need MachineSet template adjustment in
  agnosticd-v2. See Open Question Q2.

### 005 — Azure Deployment (OCP 4.22 + RHOAI + GPU)  [status: planned]

- **Description:** Provision OCP 4.22 on Azure using the `openshift-cluster`
  AgnosticD deployer and apply ocp4_workload_rhoai. GPU node pool uses
  Standard_NC6s_v3 instances (1× NVIDIA V100).
- **Outcome:** Live RHOAI dashboard on Azure cluster with one V100 GPU node.
  agnosticd_user_info emitted. Deployment teardown confirmed clean.
- **Scope (in):** vars/rhoai-azure.yml provision, GPU MachineSet
  (Standard_NC6s_v3, 1 replica).
- **Scope (out):** AWS and GCP deployments.
- **Depends on:** 002
- **Governed by:** C-01, C-02, C-03, C-04
- **Spec dir:** specs/005-azure-deployment

### 006 — OCP 5 Support  [status: planned]

- **Description:** Extend the deployment to OCP 5 once a stable AgnosticD
  installer config exists. Add rhoai-aws-ocp5.yml and equivalent GCP/Azure vars
  files; verify rhai-on-openshift-chart compatibility.
- **Outcome:** RHOAI deployable on OCP 5 clusters on all three cloud providers
  using the same ocp4_workload_rhoai role with updated installer version vars.
- **Scope (in):** OCP 5 vars files for AWS/GCP/Azure; installer version bump;
  compatibility verification with rhai-on-openshift-chart.
- **Scope (out):** New RHOAI features specific to OCP 5; breaking changes to the
  OCP 4.22 path.
- **Depends on:** 003
- **Governed by:** C-01
- **Spec dir:** specs/006-ocp5-support
- **Notes:** Blocked on confirmation that the agd CLI supports an OCP 5 installer
  config. Revisit when OCP 5 GA and AgnosticD support are confirmed.

## Open Questions

- **Q1** — Does rhai-on-openshift-chart v3.4 require the RHOAI operator channel
  to be `stable` instead of `beta` on OCP 4.22? The channel is set to `beta` in
  odh-gitops values.yaml. Resolution: check the chart's OLM Subscription template
  when deploying spec 003.
- **Q2** — Does ocp4_workload_machinesets support the GCP `accelerator` field for
  GPU node pools, or does it require a custom MachineSet manifest? Resolution:
  verify during spec 004 against the agnosticd-v2 MachineSet role implementation.

## Cross-Cutting Notes

- All three cloud deployments (003, 004, 005) share the same ocp4_workload_rhoai
  role and differ only in the cloud-provider vars file. A regression in the role
  affects all three.
- Secrets (pull secret, cloud credentials) are stored outside the repo in
  `~/agnosticd-v2-secrets/` and are never committed. The vars/secrets-template.yml
  provides the structure.
- The `rhoai-devtest` branch of odh-gitops is used in place of `rhoai-3.4`
  (C-05). If `rhoai-3.4` appears upstream in the future, evaluate a branch switch
  before spec 003.

---

**Version**: 1.0.0 | **Ratified**: 2026-11-24 | **Last Amended**: 2026-11-24

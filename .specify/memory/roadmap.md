<!--
SYNC IMPACT REPORT
==================
Version change: 1.1.2 → 1.2.0
Bump rationale: MINOR — spec 004 (GCP) in-progress with significant findings.
  OCP 4.22.22 deployed on GCP (us-east1). RHOAI operator, ArgoCD, DSC, dashboard
  all operational. GPU node attached but no hardware GPU visible (GCP Machine API
  OCP 4.22 does not pass guestAccelerators; g2/L4 not available in us-east1).
  Q2 resolved: GCP GPU requires dedicated machine type (g2) in a supported region.

Changes this revision:
  - Status: 004 planned → in-progress
  - Spec 004 notes updated with deployment lessons
  - C-07 added: GCP GPU region constraint
  - Q2 resolved

Specs affected: 004
Open questions resolved: Q2 (GCP accelerator field) — confirmed OCP 4.22 Machine
  API does not support guestAccelerators (webhook rejects field as unknown). GPU
  nodes on GCP require g2-standard-4 (L4) machine type in a region that supports
  g2 (us-east4, us-central1 — NOT us-east1). Deploy GCP cluster in us-east4 for
  GPU support.

Notes: GCP cluster live at rhoai01gcp.txdmd.gcp.redhatworkshops.io with 9 nodes.
  RHOAI dashboard accessible HTTP 200. GPU hardware not allocatable (region constraint).
  v0.2.0-gcp release gate blocked on GPU check. Next: redeploy in us-east4.

--- Prior revision (1.1.2, 2026-11-26) ---
Version change: 1.0.3 → 1.1.1
Bump rationale: MINOR — added spec 008 (RHOAI Post-Deploy Validation) + 003 in-progress.
Changes: Status: 003 planned → in-progress; spec 008 added.

--- Prior revision (1.1.0, 2026-11-25) ---
Version change: 1.0.3 → 1.1.0
Bump rationale: MINOR — vision clarification (user-workload framing) + spec 007.
Changes: Vision updated; spec 007 added.

--- Prior revision (1.0.2, 2026-11-25) ---
Version change: 1.0.1 → 1.0.2
Bump rationale: PATCH — status transition: 002 implemented → verified.
  Stale ansible-lint note in 002 entry updated to reflect passing lint.
  Debrief report debrief-2026-11-24T3.md confirmed ✅ PROCEED (0 Must-Address
  findings, 8/8 tasks checked, ansible-lint production profile pass,
  dep 001 satisfied).
Changes: Status: 002 implemented → verified; 002 notes updated.
Notes: 002 verified on 2026-11-25. Unblocked 003, 004, and 005.

--- Prior revision (1.0.1, 2026-11-24) ---
Version change: 1.0.0 → 1.0.1
Bump rationale: PATCH — status transition only: 001 implemented → verified.
  Debrief report debrief-2026-11-24T2.md confirmed ✅ PROCEED (0 Must-Address
  findings, 5/5 tasks checked, no dependencies, no diagnostics).
Changes: Status: 001 implemented → verified.
Notes: 001 verified on 2026-11-24. Cleared SS_DEPENDENCY_BLOCKED on 002.

--- Prior revision (1.0.0, 2026-11-24) ---
Version change: (none) → 1.0.0
Bump rationale: MINOR — initial formal issue; promoted from hand-written
  minimal format to full template structure. All six entries enriched with
  description, outcome, scope, notes, and constraints. No entry removed
  or restated.
Changes: Promoted hand-written roadmap; added Vision & End States;
  Constraints & Decisions C-01 through C-06; enriched all six entries;
  Open Questions Q1 and Q2; Cross-Cutting Notes.
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

- **Primary goal:** RHOAI serves as a managed AI/ML platform that end users log
  into and run their own workloads — notebooks, pipelines, model serving — on
  GPU-enabled OCP clusters. The provisioning pipeline is the means; a working,
  user-accessible RHOAI environment is the end.
- Engineers and Red Hat field teams can provision a fully operational RHOAI
  environment on OCP 4.22 — with GPU support — on AWS, GCP, or Azure by running
  a single `agd provision` command against a cloud-provider vars file.
- Each cloud-provider deployment is a discrete, verifiable spec tracked by
  speckit-roadmap: a spec unblocks only after its upstream role is recorded
  verified.
- The pinned OCP and RHOAI versions are tracked by CI: a weekly workflow detects
  drift against new upstream releases and opens a GitHub issue so maintainers can
  decide whether to update and re-validate.
- OCP 5 support is planned and unblocks when AgnosticD confirms stable installer
  support for OCP 5.

## Constraints & Decisions

- **C-07 — GCP GPU region requirement:** g2-standard-4 (NVIDIA L4) is NOT
  available in us-east1. For GCP GPU deployments, use us-east4 or us-central1.
  OCP 4.22's GCP Machine API webhook rejects `guestAccelerators` as an unknown
  field, so n1+T4 acceleration cannot be attached via MachineSet. Only dedicated
  GPU machine types (g2-standard-4) work. Set `gcp_region: us-east4` in
  vars/rhoai-gcp.yml for the next deployment.
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

### 001 — AgnosticD Infra Project Scaffold  [status: verified]

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

### 002 — ocp4_workload_rhoai Ansible Role  [status: verified]

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
- **Notes:** ansible-lint passes at production profile (0 failures, 0 warnings,
  verified 2026-11-25). Registry credentials preflight assert added (F1 fix).
  stop/start/status lifecycle handlers added (F2 fix). `gitops_branch` default
  is `rhoai-devtest` per C-05.

### 003 — AWS Deployment (OCP 4.22 + RHOAI + GPU)  [status: verified]

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
- **Notes:** Verified 2026-11-26. Deployment-time lessons: (1) ArgoCD app-controller
  needs ClusterRoleBinding cluster-admin for DataScienceCluster + DSCInitialization
  (now in role); (2) RHOAI chart installs operators but not operands — NFD
  NodeFeatureDiscovery and GPU Operator ClusterPolicy must be created separately
  (now in role); (3) chart schema requires explicit managementState: Removed for all
  components; (4) cert-manager channel stable-v1.15 does not exist on OCP 4.22 —
  use stable-v1 (Q1 resolved); (5) bastion must use RHUI marketplace image (Hourly2)
  not GOLD image. All lessons automated in ocp4_workload_rhoai role.

### 004 — GCP Deployment (OCP 4.22 + RHOAI + GPU)  [status: in-progress]

- **Description:** Provision OCP 4.22 on GCP using the `openshift-cluster`
  AgnosticD deployer and apply ocp4_workload_rhoai. GPU node pool uses
  g2-standard-4 (NVIDIA L4, 24GB VRAM) in a region that supports g2 machines.
- **Outcome:** Live RHOAI dashboard on GCP cluster with one GPU node.
  agnosticd_user_info emitted. Deployment teardown confirmed clean.
- **Scope (in):** vars/rhoai-gcp.yml provision, GPU MachineSet (g2-standard-4).
- **Scope (out):** AWS and Azure deployments.
- **Depends on:** 002
- **Governed by:** C-01, C-02, C-03, C-04, C-07
- **Spec dir:** specs/004-gcp-deployment
- **Notes (2026-11-27):** OCP 4.22.22 deployed in us-east1 (project openenv-txdmd).
  Cluster live at rhoai01gcp.txdmd.gcp.redhatworkshops.io. All operators installed:
  cert-manager, NFD, GPU Operator, GitOps, RHOAI. ArgoCD RHOAI app Synced/Healthy.
  DataScienceCluster Ready. Dashboard HTTP 200. GPU node provisioned but no hardware
  GPU visible. Root causes confirmed:
  (1) g2-standard-4 (L4) not available in us-east1 — STOCKOUT in all 3 zones
      (actual root cause: g2 machine series not supported in us-east1 region).
  (2) n1+T4 via guestAccelerators rejected by OCP 4.22 Machine API webhook as
      unknown field — VM created without GPU.
  (3) OLM OperatorGroup must exist before Subscription — install_operator role
      creates it but needs both in same namespace block.
  (4) cert-manager GCP DNS01 solver requires gcp_credentials (JSON dict) even when
      no certificates requested — set via lookup('file', '/secrets/gcp-service-account.json') | from_json.
  (5) Workers need n2-standard-4 ×5 minimum (3 insufficient for RHOAI dashboard).
  Blocking release gate item: nvidia.com/gpu not allocatable. Resolution: re-deploy
  cluster in us-east4 (g2 supported) or us-central1 for v0.2.0-gcp release tag.

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

### 007 — CI Version-Drift Detection  [status: planned]

- **Description:** GitHub Actions scheduled workflow (weekly) that compares the
  pinned versions in the repo — OCP installer version, rhai-on-openshift-chart
  OCI tag, and odh-gitops `rhoai-devtest` branch commit SHA — against the latest
  published releases, and opens a GitHub issue when drift is detected.
- **Outcome:** The repo never silently falls behind a new OCP or RHOAI release.
  When a new version is available, a GitHub issue is opened automatically with a
  diff table showing current vs. latest, so the maintainer can decide whether to
  update and re-run spec 003. No auto-merging or auto-deploy.
- **Scope (in):** `.github/workflows/version-check.yml` scheduled weekly; OCP
  release check (`quay.io/openshift-release-dev/ocp-release` tags); RHOAI chart
  check (`registry.redhat.io/rhai` OCI tags); odh-gitops branch check (GitHub
  API last commit SHA on `rhoai-devtest`); GitHub issue creation on drift.
- **Scope (out):** Auto-bumping pinned versions; triggering an automatic re-deploy
  on version change; Dependabot-style PRs (human decision required for all bumps).
- **Depends on:** 003
- **Notes:** Pinned values in `vars/rhoai-aws.yml` (`host_ocp4_installer_version`)
  and `roles/ocp4_workload_rhoai/defaults/main.yml` (`ocp4_workload_rhoai_chart_version`)
  are the sources of truth. The workflow compares these against upstream and flags
  drift. A bump in any pinned value requires a re-run of the affected cloud
  deployment spec to verify the new version works end-to-end before closing the
  issue.
- **Spec dir:** specs/007-ci-version-drift

### 008 — RHOAI Post-Deploy Validation  [status: in-progress]

- **Description:** A cloud-agnostic Ansible role (`ocp4_workload_validate_rhoai`) that
  runs as the final workload after every RHOAI deployment (AWS, GCP, Azure) and checks
  all 8 components of the GPU + AI stack. Acts as the release gate: a deployment cannot
  be tagged as a release until all checks pass.
- **Outcome:** Every cloud deployment emits `VALIDATION: PASS` from `agnosticd_user_info`
  confirming DataScienceCluster Ready, dashboard accessible, NFD operational, GPU node
  labeled, ClusterPolicy Ready, nvidia.com/gpu allocatable, and all pods Running.
- **Scope (in):** `ocp4_workload_validate_rhoai` role; added as last workload in
  rhoai-aws.yml, rhoai-gcp.yml, rhoai-azure.yml; 8 checks with pass/fail report.
- **Scope (out):** Installing or configuring any RHOAI component (read-only).
- **Depends on:** 003
- **Governed by:** C-06
- **Spec dir:** specs/008-rhoai-validation
- **Notes:** Establishes release gate rule: verified = `VALIDATION: PASS` on live cluster.
  Tag cloud-specific releases (v0.1.0-aws, v0.2.0-gcp, v0.3.0-azure) after each passes.

## Open Questions

- **Q1** ~~Does rhai-on-openshift-chart v3.4 require the RHOAI operator channel
  to be `stable` instead of `beta` on OCP 4.22?~~ **RESOLVED 2026-11-26:** channel `stable-v1`
  is correct on OCP 4.22. cert-manager channel stable-v1.15 does not exist — use stable-v1.
- **Q2** ~~Does ocp4_workload_machinesets support the GCP `accelerator` field for
  GPU node pools, or does it require a custom MachineSet manifest?~~ **RESOLVED 2026-11-27:**
  OCP 4.22 GCP Machine API webhook rejects `guestAccelerators` as unknown field — it is not
  supported. GPU nodes on GCP require a dedicated GPU machine type (g2-standard-4 for L4).
  g2 is not available in us-east1. Next GCP deployment must use us-east4 or us-central1.
  ocp4_workload_machinesets has no GCP implementation at all; GPU MachineSet is created
  directly via kubernetes.core.k8s in ocp4_workload_rhoai role using a worker MS as template.
- **Q3** — Does the RHOAI Helm chart v3.4 require specific DSC component managementState
  values to work correctly? **Partially resolved:** Setting all components to `Removed` via
  chart values disables the RHOAI dashboard. `dashboard` and `workbenches` must be `Managed`
  for a usable deployment. The chart schema validation warnings for other component fields
  are non-fatal and can be suppressed.

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

**Version**: 1.2.0 | **Ratified**: 2026-11-24 | **Last Amended**: 2026-11-27

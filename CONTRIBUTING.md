# Contributing to openshift-ai-bootstrap

Thank you for your interest in contributing! This project uses the
**Spec Kit** workflow — contributions are spec-driven, tracked in the
roadmap, and validated before merge.

---

## Ways to contribute

- **Bug reports** — open a GitHub issue with the deployment log and cluster version
- **New cloud provider support** — specs 004 (GCP) and 005 (Azure) are open
- **GPU instance additions** — update `docs/gpu-sizing.md` and `vars/`
- **Workload roles** — new `ocp4_workload_*` roles following the AgnosticD convention
- **Documentation** — README, GPU sizing guide, troubleshooting table
- **Validation checks** — additional checks in `ocp4_workload_validate_rhoai`

---

## Spec Kit workflow

This project tracks all planned work in a speckit-roadmap. Before writing code,
the work should be captured as a spec.

### 1. Check the roadmap

```bash
spec status   # see current roadmap state
```

If your contribution maps to an existing spec (004, 005, 007, 008), check
`specs/00N-*/` for the spec.md, plan.md, and tasks.md.

### 2. Add a new spec (for new features)

New features need a spec entry. Use the roadmap write skill or manually:

```bash
mkdir -p specs/0NN-your-feature
# Create spec.md, plan.md, tasks.md following the pattern in specs/003-aws-deployment/
```

Add the entry to `.specify/memory/roadmap.md` following the existing format.

### 3. Implement

Work through the tasks in `specs/0NN-*/tasks.md`, ticking each as it completes.
Follow the AgnosticD role conventions for Ansible:

- Role names: `ocp4_workload_*`
- Variable prefix: `ocp4_workload_<rolename>_`
- All tasks must have `name:` fields
- Use FQCN for modules: `ansible.builtin.*`, `kubernetes.core.*`, `agnosticd.core.*`
- Run `ansible-lint` and confirm production profile passes before submitting

### 4. Validate

The `ocp4_workload_validate_rhoai` role runs at the end of every deployment.
For new cloud providers, all 8 checks must pass on a live deployment before
the spec can be recorded `verified`.

### 5. Open a pull request

```bash
git checkout -b feat/your-feature
# ... make changes ...
git push origin feat/your-feature
# Open PR on GitHub
```

**PR requirements:**
- Link to the spec entry in `.specify/memory/roadmap.md`
- All changed Ansible roles pass `ansible-lint` (production profile)
- If adding a cloud provider: a live deployment must have produced `VALIDATION: PASS`
- Update `docs/gpu-sizing.md` if adding new instance types
- Update `README.md` if the deployment flow changes

---

## Adding a new cloud provider (GCP or Azure)

The pattern to follow is spec 003 (AWS). The key files are:

```
vars/rhoai-<cloud>.yml          — deployment config
specs/00N-<cloud>-deployment/   — spec, plan, tasks
```

The `ocp4_workload_rhoai` and `ocp4_workload_validate_rhoai` roles are
cloud-agnostic — only the vars file changes per provider.

**GPU instance for new providers:**
- Set `ocp4_workload_validate_rhoai_gpu_instance_type`
- Set `ocp4_workload_validate_rhoai_gpu_vendor_pci_id` (10de=NVIDIA, 1002=AMD)
- Set `ocp4_workload_validate_rhoai_gpu_min_count`

See `docs/gpu-sizing.md` for recommended instance types.

---

## Code of conduct

Be respectful and constructive. This project follows the
[Contributor Covenant](https://www.contributor-covenant.org/) v2.1.

---

## Questions?

Open a GitHub issue or start a Discussion. Check `deployment_info.txt` for
cluster details if reporting a deployment issue.

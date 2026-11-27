# Dynamic Resource Allocation (DRA) — Use Cases for RHOAI

> **Status:** Reference documentation only. DRA is not enabled in the default
> `ocp4_workload_rhoai` deployment. This doc captures when and why you would
> add it in a future iteration.

---

## What is DRA?

Dynamic Resource Allocation (KEP-3063) is a Kubernetes 1.32+ feature (GA in OCP 4.22+)
that replaces the coarse `nvidia.com/gpu: 1` extended-resource model with a structured,
per-claim resource API. Instead of pods requesting whole GPUs as opaque integers,
workloads declare a `ResourceClaim` that specifies exactly what GPU capability they need
(a MIG slice, a time-slice share, a whole device). The GPU Operator DRA plugin
arbitrates those claims against available hardware.

**How it differs from the current setup:**

| | Current (extended resource) | DRA |
|---|---|---|
| Resource unit | `nvidia.com/gpu: N` (whole GPUs) | ResourceClaim with specific profile |
| Isolation | Full GPU per pod | MIG slice or time-share |
| Sharing | No — 1 GPU per pod | Yes — multiple pods per GPU |
| Scheduler awareness | Opaque integer | Structured (profile, memory, model) |
| OCP requirement | OCP 4.x (all versions) | OCP 4.22+ (Kubernetes 1.32) |

---

## Use Case 1: MIG Partitioning (A100 / H100)

Multi-Instance GPU (MIG) splits one physical A100 or H100 into up to 7 isolated
instances, each with its own VRAM, compute engines, and PCIe bandwidth. Isolation
is hardware-enforced — a runaway notebook in one MIG instance cannot affect another.

**Example:** 1× A100 40GB → 7× `MIG-1g.5gb` (5 GB each). Seven RHOAI workbench
users each get a dedicated 5 GB GPU slice. No configuration leakage between users.

| A100 40GB MIG profiles | Instances | VRAM each | Good for |
|---|---|---|---|
| `1g.5gb` | 7 | 5 GB | Small fine-tuning, exploration |
| `2g.10gb` | 3 | 10 GB | 3B–7B inference |
| `3g.20gb` | 2 | 20 GB | 7B–13B inference |
| `4g.20gb` | 1 | 20 GB | Single-user, full perf |
| `7g.40gb` | 1 | 40 GB | Whole GPU (no MIG) |

DRA enables RHOAI workbench pods to request a specific MIG profile via
`ResourceClaimTemplate` in the notebook's PodSpec, so the scheduler places
them on nodes that have the requested slice available.

**GPU support:** A100 (40GB and 80GB), H100 (80GB SXM/PCIe). T4, A10G, and L4
do **not** support MIG.

---

## Use Case 2: Time-Slicing

Time-slicing multiplexes one GPU across multiple pods by context-switching between
them (similar to CPU multitasking). Unlike MIG, there is **no memory isolation** —
all pods share the full GPU memory and a runaway job can OOM the node.

**When to use:** Dev/test environments where contention is low, GPU hardware is
pre-MIG generation (T4, A10G, L4), and cost savings outweigh isolation needs.
Not recommended for production multi-tenant workloads.

**Configuration concept (GPU Operator ClusterPolicy):**
```yaml
devicePlugin:
  config:
    name: time-slicing-config
    default: any
  sharing:
    timeSlicing:
      replicas: 4   # 4 virtual GPUs per physical GPU
```

With DRA, time-slicing requests become structured ResourceClaims rather than
oversubscribed `nvidia.com/gpu` counts.

**GPU support:** T4, A10G, L4, A100, H100 — all support time-slicing.

---

## Use Case 3: MIG + DRA for RHOAI Workbenches

The most compelling RHOAI production scenario: a team shares an A100 node, with
different workbench profiles requesting different MIG slices via DRA.

```
A100 40GB node
├── Data scientist A → ResourceClaim(profile: 3g.20gb) → 20 GB slice
├── Data scientist B → ResourceClaim(profile: 2g.10gb) → 10 GB slice
└── Data scientist C → ResourceClaim(profile: 2g.10gb) → 10 GB slice
```

Each workbench pod lists a `ResourceClaimTemplate` referencing the GPU Operator's
`ResourceClass`. The RHOAI workbench controller (or a custom PodDefaultMutatingWebhook)
injects the claim reference automatically based on the workbench size tier selected
in the RHOAI dashboard.

---

## When You DON'T Need DRA

The default `ocp4_workload_rhoai` deployment (this project) targets single-user
workshop and demo environments. Each GPU node has one workbench pod, there is no
multi-tenant contention, and the whole GPU is dedicated to that session. The current
`ClusterPolicy + NFD + nvidia.com/gpu: 1` setup is correct for this use case.

Add DRA complexity only when:
- Multiple users share the same GPU node simultaneously
- Users need VRAM isolation between sessions (not just soft limits)
- You are running A100/H100 hardware and want to maximize utilization

---

## When You DO Need DRA

| Scenario | DRA benefit |
|---|---|
| Multi-tenant RHOAI cluster, shared A100 | MIG slices prevent user interference |
| Mixed model sizes (3B and 70B) on same node | Different MIG profiles per request |
| Cost optimization on expensive A100/H100 | 7 users per GPU instead of 1 |
| Dev/test on T4/L4 with low contention | Time-slicing (no MIG needed) |
| Fine-tuning jobs + inference on same node | MIG isolates training from serving |

---

## How to Enable DRA (Future Iteration — High Level)

No Ansible changes are needed in `ocp4_workload_rhoai` to enable DRA at the
cluster level. The GPU Operator ClusterPolicy enables it via the `devicePlugin`
configuration. The key steps:

1. **Enable MIG mode on the node** (A100/H100 only):
   ```yaml
   # In ClusterPolicy
   mig:
     strategy: single   # or 'mixed' for heterogeneous profiles
   ```

2. **Install the DRA plugin** (GPU Operator handles this):
   ```yaml
   # In ClusterPolicy
   driver:
     dra:
       enabled: true
   ```

3. **Create a ResourceClass** per MIG profile:
   ```yaml
   apiVersion: resource.k8s.io/v1alpha2
   kind: ResourceClass
   metadata:
     name: gpu-mig-1g5gb
   driverName: gpu.nvidia.com
   parametersRef:
     name: mig-1g5gb-params
   ```

4. **RHOAI workbench pods request a claim** via `ResourceClaimTemplate` in
   the notebook server's PodSpec — typically injected by a PodDefault or
   mutating webhook so users don't need to write YAML.

This is tracked as a potential extension to spec 002 (`ocp4_workload_rhoai` role)
or a future spec 009, gated on Q4 resolution in the roadmap.

---

## References

- [Kubernetes DRA documentation](https://kubernetes.io/docs/concepts/scheduling-eviction/dynamic-resource-allocation/)
- [NVIDIA GPU Operator DRA support](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/latest/gpu-operator-dra.html)
- [MIG user guide — A100](https://docs.nvidia.com/datacenter/tesla/mig-user-guide/)
- [Red Hat OpenShift AI — GPU configuration](https://docs.redhat.com/en/documentation/red_hat_openshift_ai)

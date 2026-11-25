# GPU Sizing Guide for RHOAI Deployments

This guide maps GPU instance types (AWS, GCP, Azure) to the LLM and AI workload
sizes they can serve in Red Hat OpenShift AI. GPU VRAM is the primary constraint —
it determines the maximum model size that can be loaded for inference.

---

## GPU VRAM vs Model Size

| VRAM    | Models that fit (FP16/BF16) | With 4-bit quantization (GPTQ/AWQ) |
|---------|-----------------------------|------------------------------------|
| 16 GB   | Up to ~7B params            | Up to ~20B params                  |
| 24 GB   | Up to ~13B params           | Up to ~34B params                  |
| 40 GB   | Up to ~20B params           | Up to ~65B params                  |
| 80 GB   | Up to ~40B params           | Up to ~180B params                 |
| 2×80 GB | Up to ~70B params           | Up to ~180B+ params                |

**Rule of thumb:** 1B parameters ≈ 2 GB VRAM (FP16). Quantization (4-bit) reduces
this by ~4×. For inference with KV-cache, add 20–30% overhead.

**Common RHOAI model sizes:**
- `granite-7b`, `mistral-7b`, `llama-3.1-8b` → 16 GB GPU minimum
- `llama-3.1-13b`, `codellama-13b` → 24 GB GPU
- `llama-3.1-70b` → requires 2×40 GB or 1×80 GB (or quantized on 40 GB)
- `granite-20b-code` → 40 GB GPU

---

## AWS GPU Instances (us-east-2)

| Instance | GPU | VRAM | vCPU | RAM | $/hr (on-demand) | Best for |
|---|---|---|---|---|---|---|
| `g4dn.xlarge` | 1× NVIDIA T4 | 16 GB | 4 | 16 GB | ~$0.53 | 7B models, dev/test |
| `g4dn.2xlarge` | 1× NVIDIA T4 | 16 GB | 8 | 32 GB | ~$0.75 | 7B models, more CPU RAM |
| `g4dn.12xlarge` | 4× NVIDIA T4 | 64 GB | 48 | 192 GB | ~$3.91 | 13B–34B models |
| `g5.xlarge` | 1× NVIDIA A10G | 24 GB | 4 | 16 GB | ~$1.01 | 13B models |
| `g5.2xlarge` | 1× NVIDIA A10G | 24 GB | 8 | 32 GB | ~$1.21 | 13B models + more RAM |
| `g5.12xlarge` | 4× NVIDIA A10G | 96 GB | 48 | 192 GB | ~$5.67 | 70B quantized |
| `p3.2xlarge` | 1× NVIDIA V100 | 16 GB | 8 | 61 GB | ~$3.06 | 7B, high-memory workloads |
| `p3.8xlarge` | 4× NVIDIA V100 | 64 GB | 32 | 244 GB | ~$12.24 | 34B+ models |
| `p4d.24xlarge` | 8× NVIDIA A100 (40GB) | 320 GB | 96 | 1152 GB | ~$32.77 | 70B+ full precision |
| `p4de.24xlarge` | 8× NVIDIA A100 (80GB) | 640 GB | 96 | 1152 GB | ~$40.96 | 180B+ models |

**Recommended for starting out:** `g5.xlarge` (A10G, 24GB) — best price/performance
for 7B–13B models. Better than T4 for newer models due to A10G architecture.

**To use in this project:** set in `vars/rhoai-aws.yml`:
```yaml
ocp4_workload_machinesets_machineset_groups:
  - name: gpu
    instance_type: g5.xlarge   # or g5.2xlarge, g5.12xlarge
    total_replicas: 1
```

---

## GCP GPU Instances (us-east1)

| Instance | GPU | VRAM | vCPU | RAM | $/hr (on-demand) | Best for |
|---|---|---|---|---|---|---|
| `n1-standard-4` + T4 | 1× NVIDIA T4 | 16 GB | 4 | 15 GB | ~$0.57 | 7B models |
| `n1-standard-8` + T4 | 1× NVIDIA T4 | 16 GB | 8 | 30 GB | ~$0.73 | 7B models + more RAM |
| `n1-standard-4` + A100 (40G) | 1× NVIDIA A100 | 40 GB | 4 | 15 GB | ~$2.93 | 13B–20B models |
| `a2-highgpu-1g` | 1× NVIDIA A100 (40GB) | 40 GB | 12 | 85 GB | ~$3.67 | 20B+ models |
| `a2-highgpu-2g` | 2× NVIDIA A100 (40GB) | 80 GB | 24 | 170 GB | ~$7.35 | 70B quantized |
| `a2-highgpu-8g` | 8× NVIDIA A100 (40GB) | 320 GB | 96 | 680 GB | ~$29.39 | 70B full precision |
| `g2-standard-4` | 1× NVIDIA L4 | 24 GB | 4 | 16 GB | ~$0.70 | 13B models, efficient |
| `g2-standard-8` | 1× NVIDIA L4 | 24 GB | 8 | 32 GB | ~$0.84 | 13B + more RAM |

**Recommended for GCP:** `g2-standard-4` (L4, 24GB) — newer architecture than T4,
comparable cost, better throughput for transformer inference.

**To use in this project:** set in `vars/rhoai-gcp.yml`:
```yaml
ocp4_workload_machinesets_machineset_groups:
  - name: gpu
    instance_type: g2-standard-4   # L4, 24GB VRAM
    total_replicas: 1
```

---

## Azure GPU Instances (eastus)

| Instance | GPU | VRAM | vCPU | RAM | $/hr (on-demand) | Best for |
|---|---|---|---|---|---|---|
| `Standard_NC4as_T4_v3` | 1× NVIDIA T4 | 16 GB | 4 | 28 GB | ~$0.53 | 7B models |
| `Standard_NC8as_T4_v3` | 1× NVIDIA T4 | 16 GB | 8 | 56 GB | ~$0.75 | 7B + memory |
| `Standard_NC6s_v3` | 1× NVIDIA V100 | 16 GB | 6 | 112 GB | ~$3.06 | 7B, V100 architecture |
| `Standard_NC12s_v3` | 2× NVIDIA V100 | 32 GB | 12 | 224 GB | ~$6.12 | 13B models |
| `Standard_NC24s_v3` | 4× NVIDIA V100 | 64 GB | 24 | 448 GB | ~$12.24 | 34B quantized |
| `Standard_NV36ads_A10_v5` | 1× NVIDIA A10 | 24 GB | 36 | 440 GB | ~$2.37 | 13B + large CPU RAM |
| `Standard_NC24ads_A100_v4` | 1× NVIDIA A100 (80GB) | 80 GB | 24 | 220 GB | ~$3.67 | 70B quantized |
| `Standard_ND96asr_v4` | 8× NVIDIA A100 (40GB) | 320 GB | 96 | 900 GB | ~$27.20 | 70B full precision |

**Recommended for Azure:** `Standard_NC4as_T4_v3` for dev/test (cheapest),
`Standard_NV36ads_A10_v5` for 13B production workloads.

**To use in this project:** set in `vars/rhoai-azure.yml`:
```yaml
ocp4_workload_machinesets_machineset_groups:
  - name: gpu
    instance_type: Standard_NC4as_T4_v3   # T4, 16GB, cheapest
    total_replicas: 1
```

---

## Upgrade Path from Current Deployment

The current deployment uses `g4dn.xlarge` (T4, 16GB). To upgrade to a larger GPU:

1. Scale the current GPU MachineSet to 0
2. Update `vars/rhoai-aws.yml` with the new instance type
3. Run `agd provision` again (machinesets workload is idempotent — creates new, removes old)

```bash
# Scale down current GPU node
oc scale machineset <name> -n openshift-machine-api --replicas=0

# Update vars/rhoai-aws.yml instance_type, then:
cd ~/agnosticd-v2
./bin/agd provision --guid rhoai01 --config openshift-cluster --account zppkh
```

---

## Model Storage Requirements (PVC sizing)

When using RHOAI model serving, models are stored on PVCs:

| Model | Approx. disk | Recommended PVC |
|---|---|---|
| 7B (FP16) | ~14 GB | 20 GB |
| 7B (4-bit quantized) | ~4 GB | 8 GB |
| 13B (FP16) | ~26 GB | 40 GB |
| 70B (4-bit quantized) | ~35 GB | 50 GB |
| 70B (FP16) | ~140 GB | 160 GB |

---

## References

- [RHOAI GPU requirements](https://docs.redhat.com/en/documentation/red_hat_openshift_ai)
- [AWS GPU instances](https://aws.amazon.com/ec2/instance-types/#Accelerated_Computing)
- [GCP GPU instances](https://cloud.google.com/compute/docs/gpus)
- [Azure GPU VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes-gpu)

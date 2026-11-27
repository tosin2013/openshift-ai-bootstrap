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

> ⚠️ **T4 deprecation:** `n1-standard-N + T4` instances are deprecated on GCP
> with end-of-support **August 1, 2027**. Use `g2-standard-4` (L4) instead.
> Also: `ocp4_workload_machinesets` has no GCP implementation — GPU MachineSets
> are created directly by `ocp4_workload_rhoai` using `kubernetes.core.k8s`.

### G2 Family — NVIDIA L4, 24 GB VRAM each (Ada Lovelace) ✅ Recommended

| Instance | GPU | VRAM | vCPU | RAM | $/hr | Best for |
|---|---|---|---|---|---|---|
| `g2-standard-4` | 1× L4 | 24 GB | 4 | 16 GB | ~$0.70 | 13B models, dev/test |
| `g2-standard-8` | 1× L4 | 24 GB | 8 | 32 GB | ~$0.85 | 13B + more CPU RAM |
| `g2-standard-24` | 2× L4 | 48 GB | 24 | 96 GB | ~$1.69 | 34B quantized |
| `g2-standard-48` | 4× L4 | 96 GB | 48 | 192 GB | ~$3.38 | 70B quantized |
| `g2-standard-96` | 8× L4 | 192 GB | 96 | 384 GB | ~$6.76 | 70B+ full precision |

### A2 Standard Family — NVIDIA A100 40 GB (Ampere)

| Instance | GPU | VRAM | vCPU | RAM | $/hr | Best for |
|---|---|---|---|---|---|---|
| `a2-highgpu-1g` | 1× A100 40GB | 40 GB | 12 | 85 GB | ~$3.67 | 20B+ models |
| `a2-highgpu-2g` | 2× A100 40GB | 80 GB | 24 | 170 GB | ~$7.35 | 70B quantized |
| `a2-highgpu-4g` | 4× A100 40GB | 160 GB | 48 | 340 GB | ~$14.69 | 70B full precision |
| `a2-highgpu-8g` | 8× A100 40GB | 320 GB | 96 | 680 GB | ~$29.39 | 180B quantized |

### A2 Ultra Family — NVIDIA A100 80 GB (Ampere)

| Instance | GPU | VRAM | vCPU | RAM | $/hr | Best for |
|---|---|---|---|---|---|---|
| `a2-ultragpu-1g` | 1× A100 80GB | 80 GB | 12 | 170 GB | ~$5.03 | 40B+ models |
| `a2-ultragpu-2g` | 2× A100 80GB | 160 GB | 24 | 340 GB | ~$10.06 | 70B full precision |

### A3 Family — NVIDIA H100 80 GB (Hopper) — us-central1 only

| Instance | GPU | VRAM | vCPU | RAM | $/hr | Best for |
|---|---|---|---|---|---|---|
| `a3-highgpu-8g` | 8× H100 SXM | 640 GB | 208 | 1872 GB | ~$87.84 | 180B+ full precision |

**Recommended start:** `g2-standard-4` (L4, 24GB, ~$0.70/hr) — 30% cheaper than
AWS `g5.xlarge` (A10G, 24GB, ~$1.01/hr) for identical VRAM. Best price/performance
for 13B model serving. Available in us-east1.

**To use in this project:** set in `vars/rhoai-gcp.yml`:
```yaml
ocp4_workload_rhoai_gcp_gpu_machine_type: g2-standard-4   # default
ocp4_workload_rhoai_gcp_gpu_replicas: 1
ocp4_workload_rhoai_gcp_gpu_zone: us-east1-b
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

## Recommended Models per GPU Tier

> **Last updated: November 2026.** This table is refreshed weekly by
> `.github/workflows/model-catalog-check.yml`, which queries the HuggingFace
> Hub API for new model versions and opens a GitHub issue when new models appear.

### Entry tier — 16–24 GB VRAM (T4, A10G, L4, g2-standard-4)

| Model | Params | VRAM (BF16) | VRAM (4-bit) | License | Notes |
|---|---|---|---|---|---|
| `ibm-granite/granite-3.3-8b-instruct` | 8B | 16 GB | 4 GB | Apache 2.0 | Multilingual, 128K ctx |
| `ibm-granite/granite-4.1-8b-instruct` | 8B | 16 GB | 4 GB | Apache 2.0 | Reasoning (thinking), 128K ctx |
| `ibm-granite/granite-4.2-8b` | 8B | 16 GB | 4 GB | Apache 2.0 | Chain-of-thought, 512K ctx, Aug 2026 |
| `meta-llama/Llama-3.1-8B-Instruct` | 8B | 16 GB | 4 GB | Llama 3.1 | Strong general purpose |
| `mistralai/Mistral-7B-Instruct-v0.3` | 7B | 14 GB | 4 GB | Apache 2.0 | Reliable baseline |
| `mistralai/Mistral-Small-3.1-24B` | 24B | 24 GB | 8 GB | Apache 2.0 | Multimodal, fits L4 exactly |

### Mid tier — 40 GB VRAM (A100 40GB, a2-highgpu-1g) ← *current GCP deployment*

| Model | Params | VRAM needed | License | Notes |
|---|---|---|---|---|
| `ibm-granite/granite-4.2-30b` | 30B | 32 GB (BF16) / 16 GB (4-bit) | Apache 2.0 | Reasoning, tool calling, 512K ctx — **Red Hat recommended** |
| `ibm-granite/granite-4.1-30b-fp8` | 30B | 30 GB (FP8) | Apache 2.0 | FP8 optimized, 128K ctx |
| `meta-llama/Muse-Glimmer-30B` | 30B | 32 GB (4-bit K-Quant) | Apache 2.0 | Agentic, multimodal (text+images), MCP, Aug 2026 |
| `mistralai/Mistral-Small-3.1-24B` | 24B | 24 GB (BF16) | Apache 2.0 | Multimodal — fits with 16 GB to spare |
| `meta-llama/Llama-3.3-70B-Instruct` | 70B | 35–40 GB (4-bit) | Llama 3.3 | Fits tight — monitor KV-cache |
| `ibm-granite/granite-20b-code-instruct` | 20B | 20 GB (BF16) | Apache 2.0 | Code-focused, comfortable fit |

> **A100 40GB sweet spot:** Granite 4.2-30B and Muse Glimmer 30B both fit at ~32 GB
> in 4-bit quantization, leaving 8 GB headroom for KV-cache. Llama 3.3-70B fits but
> needs careful batch size tuning to avoid OOM.

### High tier — 80 GB VRAM (A100 80GB, 2×A100 40GB)

| Model | Params | VRAM needed | License | Notes |
|---|---|---|---|---|
| `meta-llama/Llama-3.1-70B-Instruct` | 70B | 70 GB (BF16) | Llama 3.1 | Full precision, no quantization needed |
| `meta-llama/Llama-3.3-70B-Instruct` | 70B | 70 GB (BF16) | Llama 3.3 | Stronger reasoning than 3.1 |
| `mistralai/Mixtral-8x7B-Instruct-v0.1` | 47B active | 90 GB (BF16) | Apache 2.0 | MoE — needs 2×A100 40GB |
| `ministral/Ministral-3-14B-Reasoning` | 14B | 14 GB | Apache 2.0 | Reasoning model, fits comfortably |

### Multi-GPU tier — 160 GB+ (4×A100+, A3/H100)

| Model | Params | Min setup | Notes |
|---|---|---|---|
| `meta-llama/Muse-Spark-816B-A42B` | 816B total / 42B active (MoE) | Multi-node A100/H100 | Restricted access; not for workshops |
| `meta-llama/Llama-3.1-405B-Instruct` | 405B | 4×A100 80GB | Full precision; quantized fits 4×A100 40GB |

---

**Model serving profiles in RHOAI 3.4:**

- **Generative (vLLM)** — single LLM via KServe + vLLM runtime. OpenAI-compatible
  `/v1/chat/completions` endpoint. GPU required. Best for chat/notebook demos.
  Enable: `components.kserve.managementState: Managed` in the DataScienceCluster.
- **Standard (Legacy)** — pre-3.4 single-model REST/gRPC. For non-LLM models
  (sklearn, XGBoost, ONNX). CPU or GPU.
- **Multi-model (ModelMesh)** — high-density sharing of many small models on shared
  GPU/CPU. For MLOps pipelines with many deployed models simultaneously.

**Rules of thumb for model selection:**
- Leave 15–20% VRAM headroom for KV-cache during inference
- 4-bit quantized models (GPTQ/AWQ/K-Quant) reduce VRAM by ~4× with <5% quality loss
- Granite 4.x models (Apache 2.0) are the Red Hat-recommended default for enterprise use
- Muse Glimmer 30B is the best current agentic/multimodal option on A100 40GB (Nov 2026)

---

## Keeping the Model List Current

Model availability in the RHOAI catalog and on HuggingFace Hub changes frequently.
The workflow at `.github/workflows/model-catalog-check.yml` runs every Monday at
09:00 UTC and:

1. Queries the HuggingFace Hub API for new models from `ibm-granite` and
   `meta-llama` organizations
2. Cross-checks against the model names pinned in this document
3. Opens a GitHub issue listing any new models with their HuggingFace URLs,
   so a maintainer can evaluate whether to add them to the table above

This keeps the sizing guide current without requiring manual monitoring of
upstream model releases.

---

## References

- [RHOAI GPU requirements](https://docs.redhat.com/en/documentation/red_hat_openshift_ai)
- [AWS GPU instances](https://aws.amazon.com/ec2/instance-types/#Accelerated_Computing)
- [GCP GPU instances](https://cloud.google.com/compute/docs/gpus)
- [Azure GPU VMs](https://learn.microsoft.com/en-us/azure/virtual-machines/sizes-gpu)
- [DRA use cases for RHOAI](dra-use-cases.md)

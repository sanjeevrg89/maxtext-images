# MaxText Docker Images

Public Docker images for [MaxText](https://github.com/AI-hypercomputer/maxtext) training workloads on TPU and GPU, with ready-to-use Kubernetes manifests.

## Available Images

### TPU

| Image | Description |
|-------|-------------|
| `sanjeevrg/maxtext-tpu-pretraining:stable` | Pre-training with stable JAX |
| `sanjeevrg/maxtext-tpu-pretraining:nightly` | Pre-training with nightly JAX |
| `sanjeevrg/maxtext-tpu-posttraining:stable` | Post-training (SFT/GRPO) with stable JAX |
| `sanjeevrg/maxtext-tpu-posttraining:nightly` | Post-training (SFT/GRPO) with nightly JAX |

### GPU

| Image | Description |
|-------|-------------|
| `sanjeevrg/maxtext-gpu-pretraining:stable` | Pre-training with stable JAX |
| `sanjeevrg/maxtext-gpu-pretraining:nightly` | Pre-training with nightly JAX |

> GPU post-training is not yet supported upstream.

## Quick Start

```bash
# TPU
docker pull sanjeevrg/maxtext-tpu-pretraining:stable
docker pull sanjeevrg/maxtext-tpu-posttraining:stable

# GPU
docker pull sanjeevrg/maxtext-gpu-pretraining:stable
```

Each image is also tagged by date (`stable-2026-05-08`) and upstream commit SHA (`stable-abc1234`) for pinning.

## Kubernetes Manifests (Ironwood 4x4x8)

Ready-to-use manifests in [`k8s/`](k8s/):

| Manifest | Type | Topology | Use Case |
|----------|------|----------|----------|
| `tpu-pretraining-jobset.yaml` | JobSet | 4x4x8 (128 chips, 32 VMs) | Pre-training |
| `tpu-posttraining-jobset.yaml` | JobSet | 4x4x8 (128 chips, 32 VMs) | Post-training (SFT) |
| `tpu-inference-lws.yaml` | LeaderWorkerSet | 2x4x1 (8 chips) | Inference (vLLM) |

### Deploy Pre-Training

```bash
export WORKLOAD_NAME=maxtext-pretrain-$(date +%s)
export BASE_OUTPUT_DIR=gs://your-bucket/output
envsubst < k8s/tpu-pretraining-jobset.yaml | kubectl apply -f -
```

### Deploy Post-Training (SFT)

```bash
export WORKLOAD_NAME=maxtext-sft-$(date +%s)
export BASE_OUTPUT_DIR=gs://your-bucket/output
export LOAD_PARAMETERS_PATH=gs://your-bucket/pretrained-checkpoint
envsubst < k8s/tpu-posttraining-jobset.yaml | kubectl apply -f -
```

### Deploy Inference (vLLM)

```bash
kubectl apply -f k8s/tpu-inference-lws.yaml
kubectl port-forward svc/maxtext-vllm-svc 8000:8000
```

### Adapting to Other Topologies

| Topology | Chips | VMs (parallelism) | `google.com/tpu` per VM |
|----------|-------|--------------------|-------------------------|
| 2x2x1   | 4     | 1                  | 4                       |
| 2x2x2   | 8     | 2                  | 4                       |
| 4x4x4   | 64    | 16                 | 4                       |
| 4x4x8   | 128   | 32                 | 4                       |
| 4x8x8   | 256   | 64                 | 4                       |

Update `parallelism`, `completions`, and `cloud.google.com/gke-tpu-topology` accordingly.

## Mirror to Internal Registry

```bash
# crane (recommended)
crane copy sanjeevrg/maxtext-tpu-pretraining:stable \
  your-registry.example.com/maxtext/tpu-pretraining:stable

# skopeo
skopeo copy \
  docker://sanjeevrg/maxtext-tpu-pretraining:stable \
  docker://your-registry.example.com/maxtext/tpu-pretraining:stable
```

Both DockerHub and GitHub Container Registry support pull-through cache.

## Build Details

- **Schedule**: Weekly (Monday 00:00 UTC) from upstream `main`
- **Manual**: Workflow dispatch with configurable branch/tag and JAX mode (stable/nightly)
- **Source**: Clones [AI-hypercomputer/maxtext](https://github.com/AI-hypercomputer/maxtext) at build time. No fork required.
- **GPU patch**: Applies a fix for the upstream Ubuntu 22.04/24.04 NVIDIA repo mismatch ([details](https://github.com/AI-hypercomputer/maxtext/blob/main/src/dependencies/dockerfiles/maxtext_gpu_dependencies.Dockerfile#L11))

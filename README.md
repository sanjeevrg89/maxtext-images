# MaxText Docker Images

Public Docker images for [MaxText](https://github.com/AI-hypercomputer/maxtext) training workloads on TPU and GPU, with ready-to-use Kubernetes manifests for Ironwood (TPU v7).

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
docker pull sanjeevrg/maxtext-tpu-pretraining:stable
docker pull sanjeevrg/maxtext-tpu-posttraining:stable
docker pull sanjeevrg/maxtext-gpu-pretraining:stable
```

Each image is also tagged by date (`stable-2026-05-08`) and upstream commit SHA (`stable-abc1234`) for pinning.

## Kubernetes Manifests (Ironwood TPU v7)

Ready-to-use manifests in [`k8s/`](k8s/) targeting Ironwood (`tpu7x`, 4 chips per VM).

### Training (JobSet)

| Manifest | Topology | Chips | VMs | Use Case |
|----------|----------|-------|-----|----------|
| `tpu-pretraining-jobset.yaml` | 4x4x8 | 128 | 32 | Pre-training |
| `tpu-posttraining-jobset.yaml` | 4x4x8 | 128 | 32 | Post-training (SFT) |

### Inference (vLLM)

| Manifest | Type | Topology | Chips | VMs | Use Case |
|----------|------|----------|-------|-----|----------|
| `tpu-inference-deployment.yaml` | Deployment | 2x2x1 | 4 | 1 | Single-host, scalable replicas |
| `tpu-inference-lws.yaml` | LeaderWorkerSet | 2x2x2 | 8 | 2 | Multi-host with Ray |

### Deploy

```bash
# Pre-training
export WORKLOAD_NAME=maxtext-pretrain-$(date +%s)
export BASE_OUTPUT_DIR=gs://your-bucket/output
export MODEL_NAME=your-model-name
export PER_DEVICE_BATCH_SIZE=4.0
export MAX_TARGET_LENGTH=8192
export STEPS=30
envsubst < k8s/tpu-pretraining-jobset.yaml | kubectl apply -f -

# Inference (single-host)
export MODEL_NAME=your-model-name
envsubst < k8s/tpu-inference-deployment.yaml | kubectl apply -f -
```

### Ironwood (TPU v7) Valid Topologies

| Topology | Chips | VMs | Type |
|----------|-------|-----|------|
| 2x2x1   | 4     | 1   | Single-host |
| 2x2x2   | 8     | 2   | Multi-host |
| 4x4x4   | 64    | 16  | Multi-host |
| 4x4x8   | 128   | 32  | Multi-host |
| 4x8x8   | 256   | 64  | Multi-host |

All Ironwood VMs have 4 chips. Update `parallelism`, `completions`, and `cloud.google.com/gke-tpu-topology` accordingly.

## Mirror to Internal Registry

```bash
crane copy sanjeevrg/maxtext-tpu-pretraining:stable \
  your-registry.example.com/maxtext/tpu-pretraining:stable
```

Both DockerHub and GitHub Container Registry support pull-through cache.

## Build Details

- **Schedule**: Weekly (Monday 00:00 UTC) from upstream `main`
- **Manual**: Workflow dispatch with configurable branch/tag and JAX mode (stable/nightly)
- **Source**: Clones [AI-hypercomputer/maxtext](https://github.com/AI-hypercomputer/maxtext) at build time
- **GPU patch**: Fixes upstream Ubuntu 22.04/24.04 NVIDIA repo mismatch

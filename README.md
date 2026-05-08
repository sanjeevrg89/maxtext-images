# MaxText Docker Images

Public Docker images for [MaxText](https://github.com/AI-hypercomputer/maxtext) training workloads on TPU and GPU.

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

Each image is also tagged by date (`stable-2026-05-07`) and upstream commit SHA (`stable-abc1234`) for pinning.

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

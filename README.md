# MaxText TPU Docker Images

Public Docker images for [MaxText](https://github.com/AI-hypercomputer/maxtext) TPU training workloads.

## Available Images

| Image | Description |
|-------|-------------|
| `sanjeevrg89/maxtext-tpu-pretraining:stable` | TPU pre-training with stable JAX |
| `sanjeevrg89/maxtext-tpu-pretraining:nightly` | TPU pre-training with nightly JAX |
| `sanjeevrg89/maxtext-tpu-posttraining:stable` | TPU post-training (SFT/GRPO) with stable JAX |
| `sanjeevrg89/maxtext-tpu-posttraining:nightly` | TPU post-training with nightly JAX |

## Pull

```bash
# TPU pre-training (stable JAX)
docker pull sanjeevrg89/maxtext-tpu-pretraining:stable

# TPU post-training (stable JAX)
docker pull sanjeevrg89/maxtext-tpu-posttraining:stable
```

Each image is also tagged with date (`stable-2026-05-07`) and upstream commit SHA (`stable-abc1234`).

## Mirror to internal registry

```bash
# Using crane
crane copy sanjeevrg89/maxtext-tpu-pretraining:stable \
  your-registry.example.com/maxtext-tpu-pretraining:stable

# Using skopeo
skopeo copy \
  docker://sanjeevrg89/maxtext-tpu-pretraining:stable \
  docker://your-registry.example.com/maxtext-tpu-pretraining:stable
```

## Build schedule

- **Weekly** (Monday 00:00 UTC) from upstream `main` branch
- **Manual** via workflow dispatch with configurable branch/tag and JAX mode

## How it works

The GitHub Actions workflow clones [AI-hypercomputer/maxtext](https://github.com/AI-hypercomputer/maxtext) at build time and uses the upstream Dockerfiles directly. No fork or file copying required - images always reflect the latest upstream code.

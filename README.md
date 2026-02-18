# immich-gpu-docker

This repository runs Immich with NVIDIA GPU acceleration using Docker Compose.

## What this compose configuration does

In `docker-compose.yml`:

- `immich-server` uses `gpus: all` and:
  - `NVIDIA_VISIBLE_DEVICES=all`
  - `NVIDIA_DRIVER_CAPABILITIES=compute,video,utility`
- `immich-machine-learning` uses CUDA image `immich-machine-learning:${IMMICH_VERSION:-release}-cuda` with `gpus: all` and:
  - `NVIDIA_VISIBLE_DEVICES=all`
  - `NVIDIA_DRIVER_CAPABILITIES=compute,utility`

These settings require the host Docker engine to be configured with NVIDIA Container Toolkit.

## Prerequisites

Before starting containers, prepare:

1. Linux host with a supported NVIDIA GPU.
2. NVIDIA GPU driver installed on host (verify with `nvidia-smi`).
3. Docker Engine + Docker Compose plugin installed.
4. NVIDIA Container Toolkit installed and configured for Docker:
   - Official guide: https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
5. Create/download your `.env` file by following Immich official Docker Compose docs:
   - https://docs.immich.app/install/docker-compose/

## Required folders to create first

Based on current `.env`:

- `UPLOAD_LOCATION=/mnt/sda1/immich-data`
- `DB_DATA_LOCATION=/mnt/sda1/immich-db`

Create them before `docker compose up`:

```bash
sudo mkdir -p /mnt/sda1/immich-data /mnt/sda1/immich-db
sudo chown -R $USER:$USER /mnt/sda1/immich-data /mnt/sda1/immich-db
```

Notes:

- `model-cache` is a named Docker volume and is auto-created by Docker.
- `/etc/localtime` is mounted read-only and should already exist on normal Linux hosts.

## Install and enable NVIDIA Container Toolkit for Docker

Official documentation:
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

Follow the exact steps for your distribution from NVIDIA docs. For Ubuntu/Debian, the usual flow is:

```bash
sudo apt-get update && sudo apt-get install -y --no-install-recommends ca-certificates curl gnupg2
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg
curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
  sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
  sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update
sudo apt-get install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker
```

If you use rootless Docker, use the rootless section in the NVIDIA guide.

## Quick GPU validation in Docker

Check host GPU:

```bash
nvidia-smi
```

Check Docker GPU access:

```bash
docker run --rm --gpus all nvidia/cuda:12.4.1-base-ubuntu22.04 nvidia-smi
```

If this succeeds, Compose services with `gpus: all` can access the GPU.

## Run Immich

```bash
docker compose up -d
```

Then check container status:

```bash
docker compose ps
docker compose logs -f immich-machine-learning
```
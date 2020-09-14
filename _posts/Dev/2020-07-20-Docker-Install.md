---
title; Docker Install

categories:
  - Dev
tags:
  - Docker
---
```bash
curl -fsSL [https://get.docker.com](https://get.docker.com/) | sudo sh
docker -v #버전확인
sudo usermod -a -G docker <USER_ID> #유저 추가
```

2. **Nvidia-docker 설치 (reference: [link](https://medium.com/@sh.tsang/docker-tutorial-5-nvidia-docker-2-0-installation-in-ubuntu-18-04-cb80f17cac65))**

```bash
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd

# Test nvidia-smi with the latest official CUDA image
docker run --runtime=nvidia --rm nvidia/cuda:9.0-base nvidia-smi
```

unknown runtime 에러 나는 경우

```bash
apt-cache madison nvidia-docker2 nvidia-container-runtime

sudo apt-get install nvidia-docker2=2.0.3+docker18.03.1–1
sudo pkill -SIGHUP dockerd

# Test
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```
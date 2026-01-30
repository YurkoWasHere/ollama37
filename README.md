# Ollama Build Instructions

This repository contains Ollama, a tool for running large language models locally. Due to CentOS 7 reaching end-of-life, the original Docker build process may fail. This README provides alternative build methods that work.

## Quick Start - Using Pre-built Images

The easiest way to run Ollama is using the official Docker image:

```bash
# CPU only
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

# With NVIDIA GPU support
docker run --gpus all -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

## Building from Source

### Method 1: Simple Docker Build (Recommended)

This method uses Ubuntu-based images to avoid CentOS repository issues:

```bash
# Build the Docker image
docker build -f Dockerfile.simple -t ollama-build .

# Run the container
docker run -p 11434:11434 ollama-build serve

# Export for use on another machine
docker save ollama-build -o ollama-build.tar
gzip ollama-build.tar

# On the target machine, import and run:
gunzip -c ollama-build.tar.gz | docker load
docker run -p 11434:11434 ollama-build serve
```

### Method 2: GPU-Enabled Build

For NVIDIA GPU support, use the GPU-specific Dockerfile:

```bash
# Build with CUDA support
docker build -f Dockerfile.gpu -t ollama-gpu .

# Run with GPU access
docker run --gpus all -p 11434:11434 ollama-gpu serve
```

### Method 3: Original Build (May Fail)

The original build scripts use CentOS 7 which has reached EOL:

```bash
# This may fail due to CentOS mirror issues
./scripts/build_linux.sh

# Alternative: Try the Docker build
./scripts/build_docker.sh
```

## Prerequisites

### For CPU builds:
- Docker installed
- At least 8GB of RAM
- 10GB of free disk space

### For GPU builds:
- NVIDIA GPU with CUDA support
- NVIDIA Docker runtime (`nvidia-docker2`)
- CUDA 11.3+ compatible GPU
- At least 8GB VRAM for 7B models

### Installing NVIDIA Docker Runtime:

```bash
# Ubuntu/Debian
distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list

sudo apt-get update
sudo apt-get install -y nvidia-docker2
sudo systemctl restart docker
```

## Testing Your Build

```bash
# Check version
docker run --rm ollama-build --version

# Run a model (requires downloading first)
docker exec -it ollama ollama pull llama2:7b
docker exec -it ollama ollama run llama2:7b
```

## GPU Verification

To verify GPU is being used:

```bash
# Check GPU availability
nvidia-smi

# Check container GPU access
docker run --gpus all --rm nvidia/cuda:11.4.3-base-ubuntu20.04 nvidia-smi

# Check Ollama logs for GPU detection
docker logs ollama 2>&1 | grep -i gpu
```

## Troubleshooting

### CentOS Mirror Issues
If you encounter "Cannot find a valid baseurl for repo: base/7/x86_64", use the Ubuntu-based Dockerfiles provided (Dockerfile.simple or Dockerfile.gpu).

### GPU Not Detected
1. Ensure nvidia-docker2 is installed
2. Verify GPU with `nvidia-smi`
3. Use `--gpus all` flag when running container
4. Check CUDA compatibility with your GPU

### Build Failures
1. Ensure Docker has enough resources (memory/disk)
2. Try building with fewer parallel jobs
3. Clear Docker cache: `docker system prune -a`

## Model Requirements

| Model Size | RAM Required | VRAM Required (GPU) |
|------------|-------------|-------------------|
| 7B         | 8 GB        | 6 GB              |
| 13B        | 16 GB       | 10 GB             |
| 30B        | 32 GB       | 20 GB             |
| 70B        | 64 GB       | 40 GB             |

## Available Dockerfiles

- `Dockerfile.simple` - Basic CPU build using Ubuntu
- `Dockerfile.gpu` - NVIDIA GPU-enabled build
- `Dockerfile.ubuntu` - Alternative Ubuntu-based build
- `Dockerfile` - Original (may fail due to CentOS EOL)

## Additional Resources

- Original README: See [README-original.md](README-original.md)
- Official Ollama Documentation: https://ollama.com/
- Model Library: https://ollama.com/library
- GitHub Issues: https://github.com/ollama/ollama/issues

## License

See [LICENSE](LICENSE) file for details.
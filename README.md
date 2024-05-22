# images

Public docker images

- **whisper-base**

   - Docker image based on rocker/r2u:22.04
   - Containing software to perform transcriptions based on audio.whisper

```
docker pull ghcr.io/bnosac/whisper-base
```

- **whisper-base-cuda-11-8**

   - Docker image based on ghcr.io/bnosac/whisper-base:latest
   - Adding CUDA 11.8

```
docker pull ghcr.io/bnosac/whisper-base-cuda-11-8
```
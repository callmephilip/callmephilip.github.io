---
title: "Run Pytorch in Docker With Cuda Without Pulling Your Hair Out"
date: 2023-10-21T12:36:42+01:00
draft: false
---

This week I've been working on containerizing some Pytorch based software. Below are some notes on how to make this process more pleasant and relaxing by avoiding a few pitfalls.

> I used [Paperspace Machines](https://www.paperspace.com/machines), specifically their ML-in-a-Box which is an Ubuntu 20.04 based image that ships with Docker, CUDA and NVidia Docker already installed. Let me point you to this hopefully evergreen [SO thread](https://stackoverflow.com/questions/25185405/using-gpu-from-a-docker-container) in case your system is missing one or several of these components.

Without further ado, let's get into it

## Know your CUDA

Let's start by checking your CUDA version. You do so by running the following command:

```bash
nvcc --version
```

In my case, i got `11.7`. This is NOT the latest version and hopefully yours is more up to date. I hope Paperspace is going roll an updated version out sometime soon ([see this GH issue](https://github.com/Paperspace/ml-in-a-box/issues/13)).

## Pick your base Docker image

Now that we know which CUDA version we are running, head to [nvidia/cuda on Docker Hub](https://hub.docker.com/r/nvidia/cuda/tags) and pick an image that works for you. I went with [nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04](https://hub.docker.com/layers/nvidia/cuda/11.7.1-cudnn8-devel-ubuntu20.04/images/sha256-3c393a5b4ef7816dd95039aac3f84d05a4caf0a53ae82d3e4ea8950fac8c2788?context=explore). Note that this image targets CUDA `11.7`. If you are more up to date on your CUDA, you will need a newer version.

So far, my Docker file looks as follows:

```Dockerfile
FROM nvidia/cuda:11.7.1-cudnn8-devel-ubuntu20.04
```

## Grab your Pytorch

You might have run into some cryptic errors like

```
OSError: /home/user/miniconda/lib/python3.9/site-packages/torchaudio/lib/libtorchaudio.so: undefined symbol: _ZN3c104cuda9SetDeviceEi
```

Or something like

```
Traceback (most recent call last):
  File "/home/user/ai-voice-cloning/./src/main.py", line 11, in <module>
    from utils import *
  File "/home/user/ai-voice-cloning/src/utils.py", line 29, in <module>
    import torchaudio
  File "/home/user/miniconda/lib/python3.9/site-packages/torchaudio/__init__.py", line 1, in <module>
    from . import (  # noqa: F401
  File "/home/user/miniconda/lib/python3.9/site-packages/torchaudio/_extension/__init__.py", line 45, in <module>
    _load_lib("libtorchaudio")
  File "/home/user/miniconda/lib/python3.9/site-packages/torchaudio/_extension/utils.py", line 64, in _load_lib
    torch.ops.load_library(path)
  File "/home/user/miniconda/lib/python3.9/site-packages/torch/_ops.py", line 643, in load_library
    ctypes.CDLL(path)
  File "/home/user/miniconda/lib/python3.9/ctypes/__init__.py", line 382, in __init__
    self._handle = _dlopen(self._name, mode)
OSError: libcudart.so.12: cannot open shared object file: No such file or directory
```

As far as I can tell, this happens when you install `pytorch` packages that are not built against your CUDA version. To target proper binaries when installing packages, you can use the following command:

```Dockerfile
RUN pip install torch==2.0.1+cu117 torchvision==0.15.2+cu117 torchaudio==2.0.2 --index-url https://download.pytorch.org/whl/cu117
```

To pick the right version for your setup, head to [Pytorch installation section](https://pytorch.org/get-started/previous-versions/) and use commands in `Linux and Windows` section for the corresponding pytorch and CUDA version. There is also an interactive `Instal Pytorch` section on [Pytorch landing page](https://pytorch.org/) for the latest versions of PyTorch and CUDA.

### Run your container

There are 2 things that you want to pay attention to when running your container: targeting GPUs and adjusting shared memory available to Docker:

```bash
docker run --gpus all --shm-size 8G -it IMAGE_NAME
```

By default, Docker gets `64 Mb` of shared memory available and it's very likely that you will run through this limit pretty fast if you are running anything serious inside your container.

### Recap

I hope you find this useful and in case you got some gems to share, please [chip in here](https://github.com/callmephilip/callmephilip.github.io/issues) please

# <b>gpu4singularity</b> 
## Install GPU support in your [Singularity](http://singularity.lbl.gov/) container 

- [Guidelines for Contributing](.github/CONTRIBUTING)
- [Pull Request Template](.github/PULL_REQUEST_TEMPLATE)
- [Singularity Home](http://singularity.lbl.gov/)
- [Singularity on Biowulf](http://hpc.nih.gov/apps/singularity)

## About
_NOTE: `gpu4singularity` is depricated for most uses (except maybe compiling
against driver libs at bootstrap and some other weird stuff).  If you are using
Singularity >= 2.3, use the experimental option `--nv` to grant your containers 
GPU support at runtime._

[Singularity](http://singularity.lbl.gov) is a container platform that let's 
you "swap" out your host operating system for one that you control.  

Getting software within a container to recognize and use GPU hardware on the 
host can be tricky.  gpu4singularity simplifies this process by automating the 
steps you must take to install GPU support within your container.  Then you can
use programs that rely on [CUDA](https://developer.nvidia.com/cuda-zone) and 
[cuDNN](https://developer.nvidia.com/cudnn).

gpu4singularity does the following:

- Downloads an NVIDIA .run script for driver installation 
- Extracts the contents (doesn't actually install the driver)
- Moves all of the libraries to a central location (`/usr/local/nvidia`)
- Creates a bunch of symbolic links
- Edits and exports the $PATH and $LD_LIBRARY_PATH to point to the correct 
libraries and binaries 

## Installation

To use gpu4singularity, just download or copy it into your container and run 
it.  You could either shell into an existing container as the root user and do
something like this:

```
wget gpu4singularity 
chmod u+rwx gpu4singularity
./gpu4singularity
rm gpu4singularity
```

Or you could put the same lines of code into a Singularity definition file. For
instance, if you wanted to run [TensorFlow](https://www.tensorflow.org/) 
within a Singularity container with GPU support, you could start with a 
[NVIDIA CUDA/cuDNN Docker Hub image](https://hub.docker.com/r/nvidia/cuda/), run gpu4singularity 

```
BootStrap: docker
From: nvidia/cuda:8.0-cudnn5-devel

%post

    # add universe repo and install some packages
    sed -i '/xenial.*universe/s/^#//g' /etc/apt/sources.list
    export LANG=C
    apt-get -y update
    apt-get -y install vim wget perl python python-pip python-dev

    # download and run NIH HPC NVIDIA driver installer
    wget https://raw.githubusercontent.com/NIH-HPC/gpu4singularity/master/gpu4singularity
    chmod u+rwx gpu4singularity
    export VERSION=375.66
    ./gpu4singularity --verbose \
        -u http://us.download.nvidia.com/XFree86/Linux-x86_64/"${VERSION}"/NVIDIA-Linux-x86_64-"${VERSION}".run \
        -V "${VERSION}"
    rm gpu4singularity

    # install tensorflow
    pip install --upgrade pip
    pip install tensorflow-gpu
```



# <b>gpu4singularity</b> 
## Install GPU support in your [Singularity](http://singularity.lbl.gov/) container 

- [Guidelines for Contributing](.github/CONTRIBUTING)
- [Pull Request Template](.github/PULL_REQUEST_TEMPLATE)
- [Singularity Home](http://singularity.lbl.gov/)
- [Singularity on Biowulf](http://hpc.nih.gov/apps/singularity)

## About

[Singularity](http://singularity.lbl.gov) is a container platform that let's 
you "swap" out your host operating system for one that you control.  

Getting software within a container to recognize and use GPU hardware on the 
host can be tricky.  gpu4singularity simplifies this process by automating the 
steps you must take to install GPU support within your container.  Then you can
use programs that rely on CUDA and cuDNN.

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
instance, if you wanted to run [TensorFlow]() within a Singularity container
with GPU support, you could start with a [NVIDIA CUDA/cuDNN Docker Hub image](), run gpu4singularity 

```
BootStrap: docker
From: nvidia/cuda:8.0-cudnn5-devel

%post

    # add universe repo and install some packages
    sed -i '/xenial.*universe/s/^#//g' /etc/apt/sources.list
    locale-gen en_US.UTF-8
    apt-get -y update
    apt-get -y install vim wget perl python python-pip python-dev

    # download and run NIH HPC NVIDIA driver installer
    wget gpu4singularity 
    chmod u+rwx gpu4singularity
    ./gpu4singularity --verbose
    rm gpu4singularity

    # install tensorflow
    pip install --upgrade pip
    pip install tensorflow-gpu
```



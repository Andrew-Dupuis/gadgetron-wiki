Intstallation of the required components for CUDA support in the Gadgetron generally requires 3 steps:

1. Installation of the CUDA driver
2. Installation of the CUDA compiler and libraries

On Ubuntu 14.04, CUDA and required drives can be found in the restricted repositories. Simply:

1. Enable multiverse and restricted packages. In /etc/apt/sources.list:

    deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty universe multiverse restricted
    deb-src http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty universe multiverse restricted
    deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty-updates universe multiverse restricted
    deb-src http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty-updates universe multiverse restricted                                                                                                  

2. Install kernel headers for your current kernel (for nvidia driver):

    sudo apt-get install linux-headers-`uname -r`                                                                                                                                                          

3. Install nvidia driver:

    sudo apt-get install nvidia-331                                                                                                                                                                        
 
4. Install nvidia cuda toolkit:

   sudo apt-get install nvidia-cuda-toolkit

On Ubuntu 12.04, the components can easily be installed using a deb packpage provided by NVIDIA:

    wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1204/x86_64/cuda-repo-ubuntu1204_5.5-0_amd64.deb
    sudo sudo dpkg -i cuda-repo-ubuntu1204_5.5-0_amd64.deb 
    sudo apt-get update
    sudo apt-get install cuda

Follow the instructions. When you are done with the installation you may
want to add the following to your `~/.bashrc` file.

    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda/lib64
    export CULA_ROOT="/usr/local/cula"
    export CULA_INC_PATH="$CULA_ROOT/include"
    export CULA_BIN_PATH_32="$CULA_ROOT/bin"
    export CULA_BIN_PATH_64="$CULA_ROOT/bin64"
    export CULA_LIB_PATH_32="$CULA_ROOT/lib"
    export CULA_LIB_PATH_64="$CULA_ROOT/lib64"
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$CULA_LIB_PATH_64    

You are now ready to compile and run CUDA (and CULA) applications. You
may want to download the CUDA SDK from Nvidia to validate your
installation but this is not required.

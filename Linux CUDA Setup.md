Intstallation of the required components for CUDA support in the Gadgetron generally requires 3 steps:

1. Installation of the CUDA driver
2. Installation of the CUDA compiler and libraries
3. Installation of CULA (linear algebra library).

On Ubuntu 12.04, the first two components can easily be installed using a deb packpage provided by NVIDIA:

    wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1204/x86_64/cuda-repo-ubuntu1204_5.5-0_amd64.deb
    sudo sudo dpkg -i cuda-repo-ubuntu1204_5.5-0_amd64.deb 
    sudo apt-get update
    sudo apt-get install cuda

Next you need to install CULA. Download cula_dense_free_R17-linux.run <http://www.culatools.com/downloads/dense/> (free registration required)

Go to the folder where the files were downloaded and type:

    chmod +x cula_dense_free_R17-linux.run 
    sudo ./cula_dense_free_R17-linux.run
 
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
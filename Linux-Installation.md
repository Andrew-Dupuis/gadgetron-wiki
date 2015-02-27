Linux is the preferred operating system to get started using the Gadgetron. All of the required dependencies are included in most major Linux distributions and can be installed easily and without having to compile anything. In the following sections we walk you through the required steps to set up a full Gadgetron installation. We assume that you are starting with a freshly installed Ubuntu 14.04 available from the Ubuntu website (<http://www.ubuntu.com>). If you don't have a machine available for installing Ubuntu, you can always try it out in a virtual machine using virtualization software such as VirtualBox (<https://www.virtualbox.org>).

If you would like to use the GPU components included in the Gadgetron and you have an Nvidia GPU available on your system, please complete the CUDA/CULA installations as described in [[Linux CUDA Setup]].

If you would like to use a RedHat6 or CentOS 6 or equivalent system, please refer to the [[Linux RHEL Installation]].

First install all dependencies for Gadgetron. The following will install everything you need:

    sudo apt-get install libhdf5-serial-dev cmake git-core \
    libboost-all-dev build-essential libfftw3-dev h5utils \
    hdf5-tools python-dev python-numpy liblapack-dev libxml2-dev \
    libxslt-dev libarmadillo-dev libace-dev python-h5py \
    python-matplotlib python-libxml2 gcc-multilib python-psutil \
    libgtest-dev nvidia-cuda-toolkit 

If you would like to use MKL (Intel Math Kernel Library), please download your installation file from Intel and do the installation. Here is what we did with MKL version 11.0.5.192:

    tar -xzvf l_mkl_11.0.5.192_intel64.tgz 
    cd l_mkl_11.0.5.192_intel64/
    sudo ./install.sh 
    
Follow the instructions and add the following paths to your `~/.bashrc`::

    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/mkl/lib/intel64

Next download, compile, and install ISMRMRD (there are more detailed instructions on the <https://github.com/ismrmrd/ismrmrd> website):

      git clone https://github.com/ismrmrd/ismrmrd.git
      cd ismrmrd/
      mkdir build
      cd build
      cmake ../
      make
      sudo make install

Last command will install the library in `/usr/local/`.

Next download the Gadgetron source code. Either obtain a [release zip file](http://gadgetron.github.io.s3-website-us-east-1.amazonaws.com/files/) or use git:

      git clone https://github.com/gadgetron/gadgetron.git

Configure and build the Gadgetron:

      cd gadgetron/
      mkdir build
      cd build
      cmake ../
      make  

Install (default location is `/usr/local/gadgetron`):

    sudo make install      

The final step is to add/modify a few environment variables in your
`~/.bashrc` file:

    export GADGETRON_HOME=/usr/local/gadgetron
    export ISMRMRD_HOME=/usr/local
    export PATH=$PATH:$GADGETRON_HOME/bin:$ISMRMRD_HOME/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GADGETRON_HOME/lib:$ISMRMRD_HOME/lib


Rename the example configuration file
`GADGETRON_HOME/config/gadgetron.xml.example` to
`GADGETRON_HOME/config/gadgetron.xml`

You are now set up to run a simple example reconstruction as outlined in [[Gadgetron Hello World]].
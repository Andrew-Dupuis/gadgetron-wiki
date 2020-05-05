***

**This guide is for Gadgetron 3.17 or older. If you are installing Gadgetron 4.0 or later, please refer to this guide in stead: [[Linux Installation (Gadgetron 4)]]**

***

Linux is the preferred operating system to get started using the Gadgetron. All of the required dependencies are included in most major Linux distributions and can be installed easily and without having to compile anything. In the following sections we walk you through the required steps to set up a full Gadgetron installation. We assume that you are starting with a freshly installed Ubuntu **18.04** available from the Ubuntu website (<http://www.ubuntu.com>). If you don't have a machine available for installing Ubuntu, you can always try it out in a virtual machine using virtualization software such as VirtualBox (<https://www.virtualbox.org>).

If you would like to use the GPU components included in the Gadgetron and you have an Nvidia GPU available on your system, please complete the CUDA/CULA installations as described in [[Linux CUDA Setup]].

If you would like to use a RedHat6 or CentOS 6 or equivalent system, please refer to the [[Linux RHEL Installation]].

First install all dependencies for Gadgetron. The following will install everything you need:

```
sudo apt-get update --quiet
sudo apt-get install --no-install-recommends --no-install-suggests --yes software-properties-common apt-utils wget build-essential cython3 emacs python3-dev python3-pip libhdf5-serial-dev cmake git-core libboost-all-dev libfftw3-dev h5utils jq hdf5-tools liblapack-dev libatlas-base-dev libxml2-dev libfreetype6-dev pkg-config libxslt-dev libarmadillo-dev libace-dev gcc-multilib libgtest-dev python3-dev liblapack-dev liblapacke-dev libplplot-dev libdcmtk-dev supervisor cmake-curses-gui neofetch supervisor net-tools cpio libpugixml-dev libopenblas-base libopenblas-dev python3-tk 

# Python packages
sudo pip3 install -U pip setuptools
sudo pip3 install numpy==1.15.4 scipy Cython tk-tools matplotlib==2.2.3 scikit-image opencv_python pydicom scikit-learn psutil pyxb lxml Pillow h5py
sudo pip3 install https://download.pytorch.org/whl/cpu/torch-1.0.0-cp36-cp36m-linux_x86_64.whl
sudo pip3 install torchvision
sudo pip3 install --upgrade tensorflow
sudo pip3 install tensorboardx visdom

```

We recommend that you use MKL (Intel Math Kernel Library). You can get a free license from [Intel](https://software.intel.com/en-us/articles/free-mkl), please obtain your license and download from Intel. Then install with:

    tar -xf l_mkl_2019.3.199.tgz 
    cd l_mkl_2019.3.199/
    sudo ./install.sh
    
Follow the instructions and add the following paths to your `~/.bashrc`::

    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/opt/intel/mkl/lib/intel64:/opt/intel/lib/intel64_lin/

Next download, compile, and install ISMRMRD (there are more detailed instructions on the <https://github.com/ismrmrd/ismrmrd> website):

      git clone https://github.com/ismrmrd/ismrmrd.git
      cd ismrmrd/
      mkdir build
      cd build
      cmake ../
      make
      sudo make install

Last command will install the library in `/usr/local/`.

Next download the Gadgetron source code. Either obtain a [release zip file](https://gadgetrondata.blob.core.windows.net/gadgetrongithubio/files/) or use git:

      git clone https://github.com/gadgetron/gadgetron.git

Configure and build the Gadgetron:

      cd gadgetron/
      mkdir build
      cd build
      cmake ../
      make  

Install (default location is `/usr/local`):

    sudo make install      

The final step is to add/modify a few environment variables in your
`~/.bashrc` file:

    export GADGETRON_HOME=/usr/local
    export ISMRMRD_HOME=/usr/local
    export PATH=$PATH:$GADGETRON_HOME/bin:$ISMRMRD_HOME/bin
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$GADGETRON_HOME/lib:$ISMRMRD_HOME/lib


Rename the example configuration file
`GADGETRON_HOME/share/gadgetron/config/gadgetron.xml.example` to
`GADGETRON_HOME/share/gadgetron/config/gadgetron.xml`

You are now set up to run a simple example reconstruction as outlined in [[Gadgetron Hello World]].

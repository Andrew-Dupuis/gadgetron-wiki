Linux is the preferred operating system to get started using Gadgetron. Most of the required dependencies are included in most major Linux distributions and can be installed easily and without having to compile anything. In the following sections we walk you through the required steps to install Gadgetron.

To follow along, you will need a reasonably modern linux distribution (Ubuntu 18.04 or Fedora 29 are covered in the instructions). 

If you would like to use the GPU components included in the Gadgetron and you have an Nvidia GPU available on your system, please complete the CUDA/CULA installations as described in [[Linux CUDA Setup]]. These components are optional, and you can proceed without them. You can add the GPU components later by completing the CUDA/CULA installations and recompiling Gadgetron.

### Installing Packaged Dependencies

Installing the packaged dependencies is usually a matter of requesting them from your package manager:

##### Ubuntu 18.04
```
sudo apt-get install build-essential \
    git-core wget make cmake gcc-multilib libgtest-dev libboost-all-dev \
    libarmadillo-dev libopenblas-dev libfftw3-dev liblapack-dev liblapacke-dev \
    libxml2-dev libxslt-dev libpugixml-dev libhdf5-dev libplplot-dev libdcmtk-dev \
    python3-dev python3-pip python3-h5py python3-scipy python3-pyxb
```

##### Fedora 29
```
sudo dnf install \
    git-core wget make cmake gcc gcc-c++ gtest-devel boost-devel boost-python3-devel \
    armadillo-devel openblas-devel fftw-devel \
    libxml2-devel libxslt-devel pugixml-devel hdf5-devel plplot-devel dcmtk-devel \
    python3-devel python3-pip python3-h5py python3-scipy python3-PyXB 
```

Installing the packaged dependencies will take some time.

### Installing Remaining Dependencies

Some Gadgetron dependencies are not currently available though package managers. Most of these are optional, and can be installed later, but for now we will need to install [ISMRMRD](https://github.com/ismrmrd/ismrmrd). The following will check out the code, compile, and then install ismrmrd.  
```
git clone https://github.com/ismrmrd/ismrmrd.git 
mkdir ismrmrd/build && cd ismrmrd/build 
cmake ..
make
sudo make install
```

To use the Gadgetron Python gadgets, you will also need the ISMRMRD Python bindings. You can install these using pip:

    sudo pip3 install ismrmrd

### Compiling and Installing Gadgetron

Download the Gadgetron source code. Either obtain a [release zip file](https://gadgetrondata.blob.core.windows.net/gadgetrongithubio/files/) or use git:

    git clone --branch Gadgetron4.0 https://github.com/gadgetron/gadgetron.git

The following commands will start the build: 

    mkdir gadgetron/build && cd gadgetron/build
    cmake ..
    make -j $(nproc)

To install (default location is `/usr/local`):

    sudo make install      

### Running Gadgetron

Once the install completes, starting Gadgetron should look something like this:

     $ gadgetron
     05-03 13:24:15.962 INFO [main.cpp:48] Running on port 9002

You are now set up to run an example reconstruction as outlined in [[Gadgetron Hello World]].

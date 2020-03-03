Linux is the preferred operating system to get started using Gadgetron. Most of the required dependencies are included in most major Linux distributions and can be installed easily and without having to compile anything. In the following sections we walk you through the required steps to install Gadgetron.

To follow along, you will need a reasonably modern linux distribution (Ubuntu 18.04 or Fedora 29 are covered in the instructions). 

If you would like to use the GPU components included in the Gadgetron and you have an Nvidia GPU available on your system, please complete the CUDA/CULA installations as described in [[Linux CUDA Setup]]. These components are optional, and you can proceed without them. You can add the GPU components later by completing the CUDA/CULA installations and recompiling Gadgetron.

### Installing Packaged Dependencies

Installing the packaged dependencies is usually a matter of requesting them from your package manager:

##### Ubuntu 18.04
```bash
sudo apt-get install build-essential \
    git-core wget make cmake gcc-multilib libgtest-dev libboost-all-dev \
    libarmadillo-dev libopenblas-dev libfftw3-dev liblapack-dev liblapacke-dev \
    libxml2-dev libxslt-dev libpugixml-dev libhdf5-dev libplplot-dev libdcmtk-dev \
    python3-dev python3-pip python3-h5py python3-scipy python3-pyxb
```

##### Fedora 29
```bash
sudo dnf install \
    git-core wget make cmake gcc gcc-c++ gtest-devel boost-devel boost-python3-devel \
    armadillo-devel openblas-devel fftw-devel \
    libxml2-devel libxslt-devel pugixml-devel hdf5-devel plplot-devel dcmtk-devel \
    python3-devel python3-pip python3-h5py python3-scipy python3-PyXB 
```

Installing the packaged dependencies will take some time.

The PyXB package is poorly maintained, and might not be available to you through the system package manager. If you find the package unavailable, you can install PyXB though pip: `sudo pip3 install PyXB`

### Installing Remaining Dependencies

Some Gadgetron dependencies are not currently available though package managers. Most of these are optional, and can be installed later, but for now we will need to install [ISMRMRD](https://github.com/ismrmrd/ismrmrd). The following will check out the code, compile, and then install ismrmrd.  
```
git clone https://github.com/ismrmrd/ismrmrd.git 
mkdir ismrmrd/build && cd ismrmrd/build 
cmake ..
make
sudo make install
```
### Compiling and Installing Gadgetron

Download the Gadgetron source code. It's probably easiest to use git:

    git clone https://github.com/gadgetron/gadgetron.git

Make a build directory and start the build: 

    mkdir gadgetron/build && cd gadgetron/build
    cmake ..
    make -j $(nproc)

To install (default location is `/usr/local`):

    sudo make install      

### Integration Tests

To verify your installation, it is advised that you run the integration tests. These tests will run your Gadgetron installation through a wide range of reconstructions, and verify the produced images. 

Currently, most of the data is provided as Siemens data files. In order to run the tests, we will need to install a conversion tool. 
```
git clone https://github.com/ismrmrd/siemens_to_ismrmrd.git 
mkdir siemens_to_ismrmrd/build && cd siemens_to_ismrmrd/build 
cmake ..
make
sudo make install
```
To download the test data, run the `get_data.py` script in the Gadgetron `test/integration` folder. The integration test data suite is pretty expansive - it's a ~10 GiB download at the time of writing. 
```
cd path/to/gadgetron/test/integration
./get_data.py 
```
Once the conversion tool is installed, and the test data downloaded, run the integration tests from the Gadgetron source folder.
```
cd path/to/gadgetron/test/integration
./run_tests.py cases/*
```
Running the integration tests will take a while, depending on your system. 

### Foreign Languages

To use the Gadgetron Python interface, you will also need the Gadgetron Python interface. You can install this using pip:

    sudo pip3 install gadgetron

To use the Gadgetron Matlab interface, you will need the Gadgetron Matlab interface. You can install this though Matlab - simply search for the `gadgetron` toolbox in the Add-Ons Manager.

### Running Gadgetron

Once the install completes, Gadgetron should be ready. Starting Gadgetron should look something like this:

     $ gadgetron
     03-03 13:16:57.586 INFO [main.cpp:49] Gadgetron 4.1.1 [3fcb3176888650bfb8075f003f37b5e936f66141]
     03-03 13:16:57.586 INFO [main.cpp:50] Running on port 9002

You are now set up to run an example reconstruction as outlined in [[Gadgetron Hello World]].

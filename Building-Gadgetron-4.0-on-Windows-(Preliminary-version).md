# Installation on Windows

## Dependencies
* [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) with C++ 
* [VCPKG](https://github.com/Microsoft/vcpkg) 
* [MKL](https://software.intel.com/en-us/mkl)
* [git](https://git-scm.com/)

## Steps
Open powershell and navigate to where vcpkg is installed
```
 $env:VCPKG_DEFAULT_TRIPLET = 'x64-windows'
.\vcpkg install boost fftw3 armadillo plplot pugixml openblas ismrmrd python3 hdf5
```

```
git clone https://github.com/gadgetron/gadgetron.git 
```

Open the folder in Visual Studio 2019. Once CMake is done, right click the CMakeLists.txt and select "CMake Settings". 

![](https://i.ibb.co/VS748xB/configure-cmake.png )

Select the USE_MKL variable in "CMake variables and cache", save and generate the cmake cache.

![](https://i.ibb.co/G2V3SpW/select-mkl.png)

Now all you need to do is build and install.


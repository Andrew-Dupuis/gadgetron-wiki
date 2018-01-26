# Introduction
# Organization of source code
# Suggested naming and definition
## Names of targets
### Libraries and toolboxes
```
target name for gadgets: gadgetron_mricore
```
```
target name for toolboxes: gadgetron_toolbox_cpucore
```
### Gadgets
GenericReconCartesianFFTGadget.h/.cpp
### Toolbox functions
```
template<typename T> EXPORTCPUCOREMATH 
void syrk(hoNDArray<T>& C, const hoNDArray<T>& A, char uplo, bool isATA);
```
or
```
template<class T> EXPORTCPUCOREMATH T stddev(hoNDArray<T>* data);
```
or
```
template<typename T> EXPORTMRICORE void coil_map_2d_Inati(const hoNDArray<T>& data, hoNDArray<T>& coilMap, size_t ks = 7, 
size_t power = 3);
```
### Toolbox Classes
```
template <typename T> class EXPORTCPUFFT hoNDFFT
```
or
```
class GadgetronTimer
```
### Standalone applications
target name is the same as cpp file name.
```
gadgetron_ismrmrd_client
```
### Matlab mex functions
Starts with "Matlab_"
```
Matlab_gt_compute_coil_map
```
## Comments
### Source files
```
/** \file   cmr_t1_mapping.h
    \brief  Implement cardiac MR t1 mapping for 2D applications
            The input has dimension [RO E1 N S SLC]
            Temporal dimension is N
    \author Hui Xue
    \brief ....
    \ author ... 
*/
```
### Comments for classes
```
    /// support wavelet transform of 1D, 2D and 3D
    template <typename T> class EXPORTCPUDWT hoNDRedundantWavelet : public hoNDWavelet<T> 
    {
    public:

        typedef typename realType<T>::Type value_type;
        typedef hoNDRedundantWavelet<T> Self;

        hoNDRedundantWavelet();
        virtual ~hoNDRedundantWavelet();

        /// these compute_wavelet_filter should be called first before calling transform
        /// transform is NOT thread-safe, since some class member variables are used as buffer for computation

        /// utility function to compute wavelet filter from commonly used wavelet scale functions
        /// wav_name : "db2", "db3", "db4", "db5"
        /// this function call will fill s_ and compute fl_d_, fh_d_, fl_r_, fh_r_
        void compute_wavelet_filter(const std::string& wav_name);
        /// compute wavelet filters from scale function
        void compute_wavelet_filter(const std::vector<T>& s);
        /// set wavelet filters
        void set_wavelet_filter(const std::vector<T>& fl_d, const std::vector<T>& fh_d, const std::vector<T>& fl_r, const std::vector<T>& fh_r);

```
### Comments for functions
```
    // the Souheil method
    // data: [RO E1 CHA], only 3D array
    // these functions are using 2D data correlation matrix
    // ks: the kernel size for local covariance estimation
    // power: number of iterations to apply power method
    template<typename T> EXPORTMRICORE void coil_map_2d_Inati(const hoNDArray<T>& data, hoNDArray<T>& coilMap, size_t ks = 7, size_t power = 3);
```

## 
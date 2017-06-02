The core reconstruction data structures and algorithms are made available through a set of toolboxes in shared libraries. The toolboxes implement the functionality of the various Gadgets, but they can also be used in standalone applications. A non-exhaustive overview of key functionality is covered in the following sections.

### <a name="sectionndarray" />NDArray

Most image processing operations involve multi-dimensional arrays. Although the Gadgetron framework does not impose any specific array structure on the user, it does come with an abstract multi-dimensional array used throughout: the [NDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html). It has a specific implementation for the CPU ([hoNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_array.html)) and GPU ([cuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_d_array.html)). Sometimes (for very large datasets) it is necessary to store the arrays in host (CPU) memory while operations on the same array should, if possible, take place on the GPU. For this kind of array we provide the [hoCuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_cu_n_d_array.html).

The abstract class definition ([NDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html)) looks like (abbreviated version):

    template <class T> class NDArray
    {
     public:
      
      NDArray ();

      virtual ~NDArray();
      
      virtual T* create(std::vector<size_t> *dimensions); 

      virtual T* create(std::vector<size_t> *dimensions, 
                        T* data, bool delete_data_on_destruct = false);
      
      inline size_t get_number_of_dimensions() const {
        return dimensions_->size();
      }

      unsigned int get_size(size_t dimension);

      boost::shared_ptr< std::vector<size_t> > get_dimensions();
      
      inline T* get_data_ptr() const { 
        return data_; 
      }
      
      inline size_t get_number_of_elements() const {
        return elements_;
      }

      // Other public functions...

    protected:

      virtual void allocate_memory() = 0;
      virtual void deallocate_memory() = 0;

      // Other private functions
      
    };

The CPU version ([hoNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_array.html)) would look like (abbreviated):

    template <class T> class hoNDArray : public NDArray<T>
    {

    public:
       // Public functions...

    protected:
       virtual void allocate_memory();
       virtual void deallocate_memory();
    };

As is seen from the [NDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html) class definition, this class has a no-argument constructor, which makes it suited for encapsulating in the [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message.html) described in [Gadgets](../Gadgetron%20Streaming%20Architecture/#gadgetslink). The procedure for creating an array with complex float values would look something like this:

    #include <hoNDArray.h>
    #include <complex>

    hoNDArray< std::complex<float> > myArray;

    std::vector<size_t> dimensions;
    dimensions.push_back(128);
    dimensions.push_back(128);

    myArray.create(&dimensions);

    // Process data

As common behaviour for all toolboxes, a runtime exception is thrown if an error is encountered. For the code snippet above, this could happen if the create method is passed an empty dimensions vector or one that exceeds the available amount of memory. The user can catch such exceptions if desired. For the streaming framework, the [Gadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget.html) base class will catch such exceptions thrown within the 'process' or 'process_config' methods.

To create an [NDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html) contained in a [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message.html) would look something like this:

    GadgetContainerMessage< hoNDArray< std::complex<float> > >* m = 
      new GadgetContainerMessage< hoNDArray< std::complex<float> > >();

    std::vector<size_t> dimensions;
    dimensions.push_back(128);
    dimensions.push_back(128);

    m->getObjectPtr()->create(&dimensions);

    // Process data or pass on to other Gadget, etc. 

    m->release(); // Delete the message block and containing data

As mentioned in [Gadgets](../Gadgetron%20Streaming%20Architecture/#gadgetslink), the [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message_base.html) is a specialized version of the [ACE\_Message\_Block](http://www.dre.vanderbilt.edu/Doxygen/Stable/libace-doc/a00393.html) class from the ACE framework. Data is passed between Gadgets in the form of ACE\_Message\_Blocks and Gadgets have access to utility functions that allow them to test if a given [ACE\_Message\_Block](http://www.dre.vanderbilt.edu/Doxygen/Stable/libace-doc/a00393.html) is in fact a particular type of [GadgetContainerMessage](https://gadgetron.github.io/api_master//class_gadgetron_1_1_gadget_container_message_base.html).

#### GPU Support

The [NDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html) data structure has a GPU implementation [cuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_d_array.html) (abbreviated version of header below):

    template <class T> class cuNDArray : public NDArray<T>
    {
     public:
      cuNDArray();

      cuNDArray(const cuNDArray<T>& a);

      // Constructor from hoNDArray
      cuNDArray(hoNDArray<T> *a);

      // Assignment operator
      cuNDArray& operator=(const cuNDArray<T>& rhs);
      
      virtual ~cuNDArray();

      virtual T* create(std::vector<unsigned int> *dimensions);

      virtual T* create(std::vector<unsigned int> *dimensions, 
                        int device_no);

      virtual T* create(std::vector<unsigned int> *dimensions, 
                        T* data, bool delete_data_on_destruct = false);

      virtual boost::shared_ptr< hoNDArray<T> > to_host() const;
      
      virtual void set_device(int device_no);
      inline int get_device() { return device_; }
      
     protected:
      
      int device_; 
      virtual void allocate_memory();
      virtual void deallocate_memory();      
    };

It has a few extra `create` functions compared to the host (CPU) version of this array. Specifically, it is possible to provide the array with the device number that the array should be allocated on. This is important when working on systems with multiple GPU processors. The default is to allocate it on the current device (device 0 unless specifically set it otherwise). It is possible to query on which device the data is allocated and to effectively move the data from one device to another through operators. Similarly, one copy constructor takes a hoNDArray and transparently copies the host data to the GPU.

#### Operators and utility functions

For both the [hoNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_array.html), [cuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_d_array.html), and [hoCuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_cu_n_d_array.html) we have defined a number of common array operations as C++ operators and overloaded functions. 

Element-wise operators for addition, subtraction, multiplication and division are defined in the header files [hoNDArray_operators.h](https://gadgetron.github.io/api_master//ho_n_d_array__operators_8h.html), [cuNDArray_operators.h](https://gadgetron.github.io/api_master//cu_n_d_array__operators_8h.html), and [hoCuNDArray_operators.h](https://gadgetron.github.io/api_master//ho_cu_n_d_array__operators_8h.html) respectively. 

Commonly used arithmetic element-wise array operations such as abs, sqrt, reciprocal, sgn, real, imag, and many others are defined in [hoNDArray_elemwise.h](https://gadgetron.github.io/api_master//ho_n_d_array__elemwise_8h.html), [cuNDArray_elemwise.h](https://gadgetron.github.io/api_master//cu_n_d_array__elemwise_8h.html), and [hoCuNDArray_elemwise.h](https://gadgetron.github.io/api_master//ho_cu_n_d_array__elemwise_8h.html) respectively.

A BLAS interface to all [NDArray]((https://gadgetron.github.io/api_master//class_gadgetron_1_1_n_d_array.html)) derived classes is also available - as well as further helper functions. Please consult the [API](https://gadgetron.github.io/api_master//files.html) file structure for an overview.

Notice that the host (CPU) implementation of many of these utilities depend on [Armadillo](http://arma.sourceforge.net/) and the device (GPU) implementation depend on [Cuda](http://developer.nvidia.com/cuda-downloads)/[Cula](http://www.culatools.com/downloads/dense/). See [Linux Installation] [Mac Installation] [Windows Installation] for installation instructions.

### vector\_td

The class [vector\_td](https://gadgetron.github.io/api_master//class_gadgetron_1_1vector__td.html) provides a basic representation of one-, two-, three-, and four-dimensional vectors (positions). It is templetized with the datatype `T` and dimensionality `D`. For convenience we provide a set of typedefs to commonly encountered instances. A subset of the definitions provided in [vector_td.h](https://gadgetron.github.io/api_master//vector__td_8h.html) is provided here (users should consult the actual class definition e.g. for additional often used constructors):

    template< class T, unsigned int D > class vector_td
    {
    public:

      T vec[D];

      __inline__ __host__ __device__ T& operator[](const int i) 
      {
        return vec[i];
      }

      __inline__ __host__ __device__ const T& operator[](const int i) const
      { 
        return vec[i];
      }
    };


    // Some typedefs for convenience

    template< class REAL, unsigned int D > struct reald{
      typedef vector_td< REAL, D > Type;
    };

    template< unsigned int D > struct intd{
      typedef vector_td< int, D > Type;
    };

    template< unsigned int D > struct uint64d{
      typedef vector_td< unsigned int, D > Type;
    };

    template< unsigned int D > struct floatd{
     typedef typename reald< float, D >::Type Type;
    };

    template< unsigned int D > struct doubled{
      typedef typename reald< double, D >::Type Type;
    };

    template< class T > struct complext{
      typedef vector_td< T, 2 > Type;
    };

A number of arithmetic and conditional operators on the [vector\_td](https://gadgetron.github.io/api_master//class_gadgetron_1_1vector__td.html) are defined in [vector_td_operators.h](https://gadgetron.github.io/api_master//vector__td__operators_8h.html). Similarly, the header [vector_td_utilities.h](https://gadgetron.github.io/api_master//vector__td__utilities_8h.html) wraps common math functionality for the [vector\_td](https://gadgetron.github.io/api_master//class_gadgetron_1_1vector__td.html) class. We encourage the reader to explore these header files to obtain an overview of the provided functionality.

The [vector\_td](https://gadgetron.github.io/api_master//class_gadgetron_1_1vector__td.html) can be used in both host and device code. As an example of use it is contained in the interface of the non-Cartesian FFT described in [Non-Cartesian FFT](#sectionnoncartesianfft).

### complext

A complex number class, [complext](https://gadgetron.github.io/api_master//class_gadgetron_1_1complext.html), which can be used in both host and device code is found in [complext.h](https://gadgetron.github.io/api_master//complext_8h.html). It contains a substantial set of useful operators and functions similarly to what is found for the [vector\_td](https://gadgetron.github.io/api_master//class_gadgetron_1_1vector__td.html) class.

### Fourier Transforms

#### Cartesian FFT

##### FFT of an hoNDArray

The Gadgetron uses the FFTW library for Cartesian fast Fourier transforms of [hoNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_array.html) structures. Users can call the FFTW directly from their code, but to make things a little easier, we provide a simple wrapper class, [hoNDFFT](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_f_f_t.html). Here is an abbreviated version:

    template <typename T> class EXPORTCPUCORE hoNDFFT
    {

    public:

     typedef std::complex<T> ComplexType;

     static hoNDFFT<T>* instance();  

     void fft(hoNDArray< ComplexType >* input, unsigned int dim_to_transform);

     void ifft(hoNDArray< ComplexType >* input, unsigned int dim_to_transform);

     void fft(hoNDArray< ComplexType >* input);

     void ifft(hoNDArray< ComplexType >* input);

    protected:
     hoNDFFT();
     virtual ~hoNDFFT();
    };

The [hoNDFFT](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_f_f_t.html) class thus provides simple wrapper functionality to perform FFTs of hoNDArrays along a specific dimension or along all dimensions. It performs *in place* FFTs and works on complex arrays of single or double precision.

An important feature of this class is that it is a process wide singleton for the Gadgetron. As outlined in the definition above, the constructor and destructor are protected and it is not possible to allocate a new FFT object. The way to use the class is through the `instance` function:

    #include "FFT.h"

    hoNDFFT<float>::instance()->fft(...);

The reason for this is that the FFTW planning routines are not thread safe. Multiple Gadgets (that each have their own thread of execution) may need to use FFTs and consequently the planning routines need to be protected with a mutex. All of this is handled inside the [hoNDFFT](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_n_d_f_f_t.html) class and since it is a singleton only one thread can run the planning routines at any given time.

As mentioned it is possible for the users to call FFTW routines directly, and there may be some performance reasons for doing so (as opposed to using this wrapper), but please be aware of this thread safety issue when you design your Gadgets. If you want to be on the safe side, use the wrapper.

##### FFT of a cuNDArray

Cartesian Fast Fourier Transform on the GPU is supported by wrapping Cuda's FFT routines with an interface similar to host version above. See the definition if the [cuNDFFT](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_d_f_f_t.html) class.

#### <a name="sectionnoncartesianfft" />Non-Cartesian FFT

A dedicated GPU-implementation of the NUFFT - often referred to a gridding - is provided. The interface is defined by the [cuNFFT_plan](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_f_f_t__plan.html) class in the header [cuNFFT.h](https://gadgetron.github.io/api_master//cu_n_f_f_t_8h.html). The class definition is provided below in abbreviated form:

    template< class REAL, unsigned int D, bool ATOMICS=false > 
    class EXPORTGPUNFFT cuNFFT_plan
    {

     public: // Main interface
        
      // Constructors
      NFFT_plan();
      NFFT_plan( typename uint64d<D>::Type matrix_size, 
                 typename uint64d<D>::Type matrix_size_os, 
                 REAL W, int device = -1 );

      // Destructor
      virtual ~NFFT_plan();

      // Clear internal storage in plan
      enum NFFT_wipe_mode { NFFT_WIPE_ALL, NFFT_WIPE_PREPROCESSING };
      bool wipe( NFFT_wipe_mode mode );

      // Replan 
      bool setup( typename uint64d<D>::Type matrix_size, 
                  typename uint64d<D>::Type matrix_size_os, 
                  REAL W, int device = -1 );
        
      // Preproces trajectory 
      // Cartesian to non-Cartesian / non-Cartesian to Cartesian / both
      enum NFFT_prep_mode { NFFT_PREP_C2NC, 
                            NFFT_PREP_NC2C, 
                            NFFT_PREP_ALL };

      bool preprocess
        ( cuNDArray<typename reald<REAL,D>::Type> *trajectory, 
          NFFT_prep_mode mode );
        
      // Execute NFFT 
      // ( Cartesian to non-Cartesian or non-Cartesian to Cartesian)  
      enum NFFT_comp_mode { NFFT_FORWARDS_C2NC, 
                            NFFT_FORWARDS_NC2C, 
                            NFFT_BACKWARDS_C2NC, 
                            NFFT_BACKWARDS_NC2C };

      bool compute( cuNDArray<complext<REAL> > *in, 
                    cuNDArray<complext<REAL> > *out, 
                    cuNDArray<REAL> *dcw, NFFT_comp_mode mode );

      // Execute NFFT iteration 
      // (Cartesian to non-Cartesian and back to Cartesian space)
      bool mult_MH_M( cuNDArray<complext<REAL> > *in, 
                      cuNDArray<complext<REAL> > *out, 
                      cuNDArray<REAL> *dcw, 
                      std::vector<unsigned int> halfway_dims );
      
     public: // Utilities
      
      // NFFT convolution 
      // (Cartesian to non-Cartesian or non-Cartesian to Cartesian)
      enum NFFT_conv_mode { NFFT_CONV_C2NC, NFFT_CONV_NC2C };
      bool convolve( cuNDArray<complext<REAL> > *in, 
                     cuNDArray<complext<REAL> > *out, 
                     cuNDArray<REAL> *dcw, 
                     NFFT_conv_mode mode, bool accumulate = false );
        
      // NFFT FFT
      enum NFFT_fft_mode { NFFT_FORWARDS, NFFT_BACKWARDS };
      bool fft( cuNDArray<complext<REAL> > *data, 
                NFFT_fft_mode mode, bool do_scale = true );
      
      // NFFT deapodization
      bool deapodize( cuNDArray<complext<REAL> > *image );

     public: // Setup queries

      typename uint64d<D>::Type get_matrix_size();
      typename uint64d<D>::Type get_matrix_size_os();
      REAL get_W();
      unsigned int get_device();
      
    ...
    };

After a [cuNFFT_plan](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_f_f_t__plan.html) is created the `preprocess` function should be called with the desired trajectory. In the special case of radial sampling the header [radial_utilities.h](https://gadgetron.github.io/api_master//radial__utilities_8h_source.html) defines some convenient functions to compute radial trajectories and corresponding density compensation weights. After preprocessing the NFFT can be executed through the `compute` function. The individual building blocks of the NFFT - convolution, FFT, and deapodization - are exposed in the public interface and hence available for use in custom algorithms.

It is often required to perform the NFFT on a number of different inputs. Particularly for small 1D and 2D arrays, the best performance is obtained if many transforms are executed concurrently in order to keep the device fully occupied. Two strategies can be combined:

-   The trajectory passed to the preprocess method is normally a one-dimension [cuNDArray](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_d_array.html) containing normalized (to the range [-0.5;0.5]) non-Cartesian positions as [reald\<REAL,D\>](https://gadgetron.github.io/api_master//struct_gadgetron_1_1reald.html) elements of precision REAL and dimensionality D. However, if the cuNDArray is two-dimensional, the latter dimension specifies that we wish to transform a number of frames with different trajectories concurrently.

-   If a number of transformations with identical trajectories are to be transformed, the input and output arrays to the compute methods can be any multiple of the Cartesian and non-Cartesian dimensions configured from the setup and preprocess methods. The images provided are consequently batch transformed.

**Please note**. The cuNFFT performs *significantly better* on GPUs supporting Cuda's shader model 2.0 or newer compared to devices supporting only shader models 1.x. The reason being that we rely on the inherent caching of global memory - available only on hardware supporting at least shader model 2.0.

A version of the cuNFFT implemented using atomic operations is available. It is enabled through the ATOMICS booloean template arguments (and defaults to false, i.e. disabled). At the time of writing, the current generation GPUs showed inferior performed using the atomic version over the non-atomic version. However, using the atomic version significantly reduces the memory requirements, and could thus be the only viable option, particularly for three- or four-dimensional reconstructions, on GPUs lacking sufficient memory to execute the non-atomic code path.

### <a name="sectionlinearmatrixoperators" />Linear (Matrix) Operators

A fundamental building block of most image reconstruction algorithms is the abstract class [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html). It inherits some functionality from the [generalOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1general_operator.html) base class. A range of linear imaging and regularization operators are derived from the pure virtual [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html).

The [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html) is templated by it underlying ARRAY\_TYPE (e.g. hoNDArray<float> or cuNDArray<float>) representing the expected vector in the matrix-vector multiplication the operator implements.

Every [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html) has an associated weight that is used to balance multiple matrix terms when added to a cost function (see [Linear Solvers](#sectionlinearsolvers)).

The main functionality is provided in the two pure virtual functions `mult_M` and `mult_MH` denoting multiplication with the matrix operator (`M`) and multiplication with the adjoint (i.e. conjugate transpose) of the matrix operator (`M`) respectively. The default implementation of `mult_MH_M` computes an "iteration" of the two (`M^H M`) by invoking `mult_M` and `mult_MH` in turn. Specialized operators can redefine the virtual `mult_MH_M` to increase performance when appropriate.

The clone method is required by some solvers to make a clone (copy) of a given linear operator. Similarly, some solvers require knowledge of the `domain` and `codomain` dimensions on which the operator can be applied. The mult\_M method converts the input vector of `domain_size` to one of `codomain_size` - and vice versa for mult\_MH.

The [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html) is used to model a linear imaging modality's encodig operation (Fourier transform for MRI, Radon transform for CT, convolution for Microscopy etc.) but also common regularization operators such the identity matrix, a partial derivative etc.

Here follows a list that briefly describes the linear operators that are used for the reconstruction examples discussed later in this document ([Gadgetron Applications], [Standalone Applications]) - inspect the [API](https://gadgetron.github.io/api_master//files.html) to obtain the full set of available operators:

#### List of linear operators

The section provides a non-exhaustive list of available linear operators in Gadgetron toolboxes.

A two-level implementation strategy is used for most of the operators the Gadgetron provide. We first derive a class, say [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html), from the [linearOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator.html). In this derived class we implement the pure virtual functions of the base class, e.g. `mult_M`, `mult_MH`, and `mult_MH_M`. The overall algorithm and functionality of the operator is implemented at this level. Like its superclass, the [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html) is however templated on the underlying ARRAY\_TYPE and thus cannot contain dedicated implementation code to a specific array implementation. The implementation of `mult_M`, `mult_MH`, and `mult_MH_M` is consequently based on a new set of pure virtual functions of the templated ARRAY\_TYPE. We provide another level of inheritance, e.g. [cuIdentityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_identity_operator.html), which in this case provides the cuNDArray-specific implementation of the pure virtual functions in the [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html). This hierarchy has the desired design goal, that the core algorithm implementation is shared in the base class of the operator. Only the host/device specific sub-components are defined individually. It is thus fairly straightforward to derive both an cuNDArray and an hoNDArray version of an operator. I.e. to build both host and device solvers with minimal effort.

As an example we provide a simplified declaration of the [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html) and [cuIdentityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_identity_operator.html) below. Without specific mentioning for the subsequent operators, they follow a similar inheritance hierarchy.

-   [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html)

    Implements multiplication of a vector with the identity matrix.

        template <class ARRAY_TYPE> class identityOperator : public linearOperator<ARRAY_TYPE>
        {
         public:

          // Simplified code without validity checks

          identityOperator() : linearOperator<ARRAY_TYPE>() {}
          virtual ~identityOperator() {}
    
          virtual void mult_M( ARRAY_TYPE *in, ARRAY_TYPE *out, bool accumulate = false )
          {
            if( accumulate )
              *out += *in;
            else 
              *out = *in;           
          }

          ... // Similar code for mul_MH
        };


    `The Cuda specific implementation:`

        // Just instantiation as there is no derived virtual functions 
        // to implement in this simple operator

        template <class T> class cuIdentityOperator : public identityOperator< cuNDArray<T> >
        {
         public:
           cuIdentityOperator() : identityOperator< cuNDArray<T> >() {}
           virtual ~cuIdentityOperator() {}
        }; 


    Notice that the template argument to the [cuIdentityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_identity_operator.html) differs from the [identityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1identity_operator.html). Type T specifies the desired type of the matrix-vector elements (e.g. float or complext<float>). Also notice how the [cuIdentityOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_identity_operator.html) class definition directly specifies the ARRAY\_TYPE of its superclass (in this case to be of type cuNDArray\<T\>).

-   [partialDerivativeOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1partial_derivative_operator.html)

    Provides the partial derivative of an image in a given spatial dimension.

-   [laplaceOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1laplace_operator.html)

    Computes the Laplacian of an image.

-   [imageOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1image_operator.html)

    Performs multiplication with a diagonal matrix of the element-wise reciprocal of a given image.

-   [convolutionOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1convolution_operator.html)

    Performs convolution of an image with a given kernel.

-   [cuNFFTOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_n_f_f_t_operator.html)

    Implements the non-Cartesian Fast Fourier Transform

-   [senseOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1sense_operator.html)

    Implements the encoding operator for the parallel MRI imaging technique Sense. Comes in two flavours for 1) [Cartesian](https://gadgetron.github.io/api_master//class_gadgetron_1_1cartesian_sense_operator.html) and 2) [non-Cartesian](https://gadgetron.github.io/api_master//class_gadgetron_1_1non_cartesian_sense_operator.html) reconstruction.

-   [multiplicationOperatorContainer](https://gadgetron.github.io/api_master//class_gadgetron_1_1multiplication_operator_contrainer.html)

    An operator can often be considered the result of multiplicative concatenation of a sequence of simpler linear operators. The multiplicationOperatorContainer defines a convenient interface to ease the construction of such concatenations.

-   [encodingOperatorContainer](https://gadgetron.github.io/api_master//class_gadgetron_1_1encoding_operator_container.html)

    As we require exactly one encoding operator (but allow multiple regularization operators) to be added to our solvers (see [Linear Solvers](#sectionlinearsolvers) below), this operator acts as a container when multiple encoding operators are desired. For example: The cost function right below ([Conjugate Gradient Method for Linear Least Squares](#sectionconjugategradient)) has two terms in its general form. Most often the vector **p** is **0** and consequently the operator **R** is considered a regularization operator while the operator **E** the single encoding operator. However, if **p** is non-zero, both **E** and **R** must be added to an encodingOperatorContainer that takes in both **m** and **p** during multiplication. A single encodingOperatorContainer is then added to the solver.

New operators are continuously added with every new release.

### <a name="sectionlinearsolvers">Linear Solvers

The Gadgetron's solvers toolbox contains a generic conjugate gradient solver to solve linear least squares reconstruction problems (see [Linear Solvers](#sectionlinearsolvers)), as well as both a non-linear conjugate gradient solver and a two flavors of a Split Bregman solver to minimize non-linear problems containing l1-norms for regularization (see [Non-Linear Solvers](#sectionnonlinearsolvers)). 

#### <a name="sectionconjugategradient" />Conjugate Gradient Method for Linear Least Squares

The conjugate gradient solver is used to reconstruct an image posed as a minimizer to an l2-based optimization problem:

![](http://gadgetron.sf.net/figs/math/lls.jpg)

The unknown image to be reconstructed is denoted here by **u** and the measured data by **m**. **E** is a linear operator modelling the encoding of the imaging modality (e.g. a Fourier transform for MRI, a Radon transform for CT etc.). **R** is a regularization operator often required to ensure uniqueness of the solution. Lambda is a scalar weight (with a default value of one) associated to each matrix operator and
used to balance the various terms in the cost function. Finally **p** denotes some (possibly blank) prior image in the regularization term. Any number of terms can be added.

The closed form solution to the optimization problem is given by the linear system of equations:

![](http://gadgetron.sf.net/figs/math/lls_form.jpg)

Put extremely short; you set up and run a solver by 1) adding the corresponding linear operators to the solver, and 2) invoking the `solve` function in the solver providing **m** (and **p** if non-zero)
as input arguments.

The interface to the conjugate solver is defined from the classes [solver](https://gadgetron.github.io/api_master//class_gadgetron_1_1solver.html), [linearOperatorSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1linear_operator_solver.html), and [cgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1cg_solver.html). Host/device specific implementations are available in the classes [hoCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_cg_solver.html), [cuCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_cg_solver.html), and [hoCuCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1ho_cu_cg_solver.html). I.e. the overall inheritance hierarchy is modelled and implemented similarly to the operator class hierarchy described above (see [Linear (Matrix) Operators](#sectionlinearmatrixoperators)). 

To use the solver, the user creates an instance of the solver for either the host or device (e.g. the [cuCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_cg_solver.html) for a GPU-based solver). The solver is then configured using the inherited functions. The core 'solve' function itself is found in the root of the hierarchy; the [solver](https://gadgetron.github.io/api_master//class_gadgetron_1_1solver.html) class.

Note that any number of terms (linear operators) can be added to the solver (or cost function).

The following code listing provides a short example of how to define a conjugate gradient solver for GPU-based image deblurring given an image and an estimate of the point spread function that degraded the image. It uses the [convolutionOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1convolution_operator.html) to model the blurring and a [partialDerivativeOperator](https://gadgetron.github.io/api_master//class_gadgetron_1_1partial_derivative_operator.html) in each spatial dimension for regularization. The full code can be found in
\$(GADGETRON\_SOURCE)`/apps/standalone/gpu/deblurring/2d/deblur_2d_cg.cpp`.

      << Code that parses the command line 
         and loads the image and kernel from disk >>

      // Define the desired precision
      typedef float _real; 
      typedef complext<_real>::Type _complext;

      // Upload host data to device
      cuNDArray<_complext> data(host_data.get());
      cuNDArray<_complext> kernel(host_kernel.get());
        
      // Setup regularization operators

      boost::shared_ptr< cuPartialDerivativeOperator<_complext,2> > Rx
       ( new cuPartialDerivativeOperator<_complext,2>(0) );

      Rx->set_weight( kappa );

      boost::shared_ptr< cuPartialDerivativeOperator<_complext,2> > Ry
       ( new cuPartialDerivativeOperator<_complext,2>(1) );

      Ry->set_weight( kappa );

      // Define encoding matrix

      boost::shared_ptr< cuConvolutionOperator<_real,2> > 
        E( new cuConvolutionOperator<_real,2>() );

      E->set_kernel( &kernel );
      E->set_domain_dimensions(data.get_dimensions().get());

      // Setup conjugate gradient solver

      cuCgSolver< _complext> cg;

      cg.set_encoding_operator( E );                         
      if( kappa>0.0 ) cg.add_regularization_operator( Rx ); 
      if( kappa>0.0 ) cg.add_regularization_operator( Ry ); 
      cg.set_max_iterations( num_iterations );
      cg.set_tc_tolerance( 1e-12 );
      cg.set_output_mode( cuCgSolver< _complext>::OUTPUT_VERBOSE );
                  
      //
      // Conjugate gradient solver
      //
  
       boost::shared_ptr< cuNDArray<_complext> > cgresult = cg.solve( &data );

      // All done, write out the result
  
      boost::shared_ptr< hoNDArray<_complext> > host_result = cgresult->to_host();
      write_nd_array<_complext>(host_result.get(), (char*)parms.get_parameter('r')->get_string_value());
    
      boost::shared_ptr< hoNDArray<_real> > host_norm = abs(cgresult.get())->to_host();
      write_nd_array<_real>( host_norm.get(), "cg_deblurred_image.real" );  


For an overview of the various standalone applications the Gadgetron provides - and instruction on how to run them - we refer to [Standalone Applications].

### <a name="sectionnonlinearsolvers" />Non-linear Solvers

#### Split Bregman Solver for L1-regularized Problems

The Gadgetron includes two Split Bregman solvers to solve respectively

![](http://gadgetron.sf.net/figs/math/sb.jpg)

where |.|<sub>TV</sub> denotes the Total Variation norm. The solver to the upper (unconstraint) optimization problem is defined in the class [sbCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1sb_cg_solver.html), while the solver to the latter constraint problem defined in the class [sbcCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1sbc_cg_solver.html). The Split Bregman solver was chosen as it integrates nicely with the linear conjugate solver described above [Linear Solvers](#sectionlinearsolvers). In fact, most of the work in the two Split Bregman solvers is performed by a linear inner solver (e.g. a conjugate gradient solver), but the input (right hand side) to the inner solver varies from iteration to iteration. 

As for the operators and conjugate gradient solvers one should instantiate either a host or device specific instance of the solver. I.e. to run the constrained Split Bregman solver on the GPU, the user would create an instance of a [cuSbCgSolver](https://gadgetron.github.io/api_master//class_gadgetron_1_1cu_sbc_cg_solver.html). Prior to running the `solve` function with the measured data **m**, the user should provide 1) the encoding operator, 2) the regularization operators, and 3) the desired domain and codomain dimensions as these cannot in general be deduced from the measured data.

We outline the code required to set up the unconstraint Split Bregman solver for TV-based image denoising. The full code can be found in \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/denoising/2d/denoise_TV.cpp`.


      << Command line parsing and data loading >>
      
      // Setup regularization operators
      // 

      boost::shared_ptr< cuPartialDerivativeOperator<_real,2> > Rx
        ( new cuPartialDerivativeOperator<_real,2>(0) );

      Rx->set_weight( lambda );
      Rx->set_domain_dimensions(data.get_dimensions().get());
      Rx->set_codomain_dimensions(data.get_dimensions().get());

      boost::shared_ptr< cuPartialDerivativeOperator<_real,2> > Ry
        ( new cuPartialDerivativeOperator<_real,2>(1) );

      Ry->set_weight( lambda );
      Ry->set_domain_dimensions(data.get_dimensions().get());
      Ry->set_codomain_dimensions(data.get_dimensions().get());

      // Define encoding operator (identity)
      //

      boost::shared_ptr< identityOperator<cuNDArray<_real> > > E
        ( new identityOperator<cuNDArray<_real> >() );

      E->set_weight( mu );
      E->set_domain_dimensions(data.get_dimensions().get());
      E->set_codomain_dimensions(data.get_dimensions().get());

      // Setup split-Bregman solver
      //

      cuSbCgSolver<_real> sb;
      sb.set_encoding_operator( E );
      sb.add_regularization_group_operator( Rx );
      sb.add_regularization_group_operator( Ry);
      sb.add_group();
      sb.set_max_outer_iterations(num_outer_iterations);
      sb.set_max_inner_iterations(num_inner_iterations);
      sb.set_output_mode( cuCgSolver<_real>::OUTPUT_VERBOSE );
  
      // Setup inner conjugate gradient solver
      //

      sb.get_inner_solver()->set_max_iterations( num_cg_iterations );
      sb.get_inner_solver()->set_tc_tolerance( 1e-4 );
      sb.get_inner_solver()->set_output_mode( cuCgSolver<_real>::OUTPUT_WARNINGS );

      // Run split-Bregman solver
      //

      boost::shared_ptr< cuNDArray<_real> > sbresult = sb.solve(&data);

      << do something with the result >>
   

The constrained Split Bregman solver inherits from the unconstaint Split Bregman solver is thus defined with a similar interface.

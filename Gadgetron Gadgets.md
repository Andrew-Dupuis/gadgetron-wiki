Gadgets wrap the functionality of the toolboxes and provide generic building blocks for configuring the streaming reconstruction in the Gadgetron.

### MRI Gadgets

One of the original motivations for creating the Gadgetron was to make a high throughput MRI reconstruction engine that could be interfaced to different MRI vendor systems. Consequently, a lot of the functionality present in the initial release toolboxes and Gadgets is focused on MRI reconstruction. In this section we review the basic data structures used to describe MRI data and list some of the MRI Gadgets that are available. These Gadgets are used in the example applications [Basic 2D FFT MRI] [Cartesian 2D Parallel MRI (GRAPPA)] [Non-Cartesian 2D Parallel MRI (SENSE)].

#### MRI Data Structures

MRI data is processed in two different phases. In the first phase individual data (k-space) acquisitions are processed while in the second phase these acquisitions have been combined into images (which may still be in k-space). Correspondingly, there are two different types of Gadgets that dominate the MRI Gadgets; those who operate on individual acquisitions and those who operate on images. Naturally, there are also transitional Gadgets that operate on acquisitions but output images.

The data header structures used by these MRI Gadgets are defined by the ISMRM Raw Data format (<http://ismrmrd.sourceforge.net>).

Most MRI Gadgets inherit from Gadget2 as described in [Gadgets](../Gadgetron%20Streaming%20Architecture/#gadgetslink), i.e. they operate on two argument types, the main two base classes used are:

    Gadget2< ISMRMRD::AcquisitionHeader, hoNDArray< std::complex<float> > >
    Gadget2< ISMRMRD::ImageHeader, hoNDArray< std::complex<float> > >

As seen, they take a data array (which is typically of complex float type) and a header describing either the acquisition or the image. These headers are defined in [ismrmrd.h](http://ismrmrd.sourceforge.net/api/ismrmrd_8h_source.html) (from the ISMRM Raw Data format). The definition of [ISMRMRD::AcquisitionHeader](http://ismrmrd.sourceforge.net/api/struct_i_s_m_r_m_r_d_1_1_acquisition_header.html) looks like (abbreviated):

    struct EncodingCounters {
     uint16_t kspace_encode_step_1; 
     uint16_t kspace_encode_step_2; 
     uint16_t average;              
     uint16_t slice;                
     uint16_t contrast;             
     uint16_t phase;                
     uint16_t repetition;           
     uint16_t set;                  
     uint16_t segment;              
     uint16_t user[8];              
    };

    struct AcquisitionHeader
    {
     uint16_t           version;                        
     uint64_t           flags;                          
     uint32_t           measurement_uid;                
     uint32_t           scan_counter;                   
     uint32_t           acquisition_time_stamp;         
     uint32_t           physiology_time_stamp[3];       
     uint16_t           number_of_samples;              
     uint16_t           available_channels;             
     uint16_t           active_channels;                
     uint64_t           channel_mask[16];               
     uint16_t           discard_pre;                    
     uint16_t           discard_post;                   
     uint16_t           center_sample;                  
     uint16_t           encoding_space_ref;             
     uint16_t           trajectory_dimensions;          
     float              sample_time_us;                 
     float              position[3];                    
     float              read_dir[3];                    
     float              phase_dir[3];                    
     float              slice_dir[3];                    
     float              patient_table_position[3];      
     EncodingCounters   idx;                            
     int32_t            user_int[8];                    
     float              user_float[8];                 
    };

It is a simple struct, which mainly serves the purpose of keeping track of a) the encoding properties of a given acquisition (phase ending number, etc.) and b) the spatial position and orientation that the data was acquired from. Different MRI systems have different conventions for how to label data, but in most cases one would be able to convert to this format.

The [ISMRMRD::ImageHeader](http://ismrmrd.sourceforge.net/api/struct_i_s_m_r_m_r_d_1_1_image_header.html) data structure is also just a struct for keeping track of image labels, position, and orientation:

    struct ImageHeader
    {
    uint16_t            version;                        
     uint64_t            flags;                         
     uint32_t            measurement_uid;               
     uint16_t            matrix_size[3];                
     float               field_of_view[3];              
     uint16_t            channels;                      
     float               position[3];                   
     float               read_dir[3];                    
     float               phase_dir[3];                    
     float               slice_dir[3];                    
     float               patient_table_position[3];     
     uint16_t            average;                       
     uint16_t            slice;                         
     uint16_t            contrast;                      
     uint16_t            phase;                         
     uint16_t            repetition;                    
     uint16_t            set;                           
     uint32_t            acquisition_time_stamp;        
     uint32_t            physiology_time_stamp[3];      
     uint16_t            image_data_type;               
     uint16_t            image_type;                    
     uint16_t            image_index;  
     uint16_t            image_series_index;
     int32_t             user_int[8];       
     float               user_float[8];     
    };

#### List of available MRI Gadgets

This section contains a non-exhaustive list of available MRI Gadgets with a few brief comments on their function. The purpose is to make it easier to read the XML configuration files provided with the Gadgetron and to give some ideas of what modules can be reused in new reconstruction programs.

-   [AccumulatorGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_accumulator_gadget.html) (`gadgetron_mricore`):

    Simple Gadget for accumulating k-space profiles in an array and passing it on to next Gadget. Used for simple Cartesian FT MRI reconstructions.

-   [AutoScaleGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_auto_scale_gadget.html) (`gadgetron_mricore`):

    Does simple histogram analysis of floating point images passing through and scales them. This is typically used upstream of conversion from floating point to unsigned short images.

-   [CoilReductionGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_coil_reduction_gadget.html) (`gadgetron_mricore`):

    Used to reduce the number of coils in a dataset. Typically used to tune the performance of a given reconstruction by eliminating data. This Gadget is commonly used in conjunction with the [PCACoilGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_p_c_a_coil_gadget.html), which generates virtual coils based on principal component analysis. The coil reduction can be specified with either a mask or the number of target coils as illustrated below

        <gadget>
         <name>CoilReduction</name>
         <dll>gadgetroncore</dll>
         <class>CoilReductionGadget</class>
         <!-- Keep a max of 16 coils -->
         <property><name>coils_out</name><value>16</value></property>
        </gadget>

        <gadget>
         <name>CoilReduction</name>
         <dll>gadgetroncore</dll>
         <class>CoilReductionGadget</class>
         <!-- Keep only coil 2,3,4,5 and discard the rest-->
         <property>
          <name>coil_mask</name>
          <value>0 1 1 1 0 0 0 0</value>
         </property>
        </gadget>

-   [CropAndCombineGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_crop_and_combine_gadget.html) (`gadgetron_mricore`):

    This Gadget is used to do a simple RMS coil combination in the image domain and remove 2x oversampling in the first dimension of the image as is commonly used in MRI. This Gadget is intended to be used after FFT of the data.

-   [ExtractGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_extract_gadget.html) (`gadgetron_mricore`):

    This Gadget is used to extract a given component (magnitude, real, imaginary, phase) from complex images, i.e. it converts complex images to real images containing specific components. The Gadget can be used to extract multiple components using a mask. The bit fields used to define the components are defined as:

        #define GADGET_EXTRACT_MAGNITUDE              (1 << 0) //1
        #define GADGET_EXTRACT_REAL                   (1 << 1) //2
        #define GADGET_EXTRACT_IMAG                   (1 << 2) //4
        #define GADGET_EXTRACT_PHASE                  (1 << 3) //8

    To specify the components, you just specify the mask, for example, the following specification would extract magnitude (1) and phase (8):

        <gadget>
         <name>Extract</name>
         <dll>gadgetroncore</dll>
         <class>ExtractGadget</class>
         <property><name>extract_mask</name><value>9</value></property>
        </gadget>

    Default behavior is to extract magnitude.

-   [FFTGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_f_f_t_gadget.html) (`gadgetron_mricore`):

    This Gadget Fourier transforms along the first 3 dimensions of the dataset (frequency, phase, partition encoding directions) and passes on the data to the next Gadget.

-   [FloatToUShortGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_float_to_u_short_gadget.html) (`gadgetron_mricore`):

    Converts floating point images to unsigned short images. This Gadget would often be used in conjunction with a scaling step (e.g. [AutoScaleGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_auto_scale_gadget.html)) upstream to ensure that the values will not get clipped or overflow during the conversion to unsigned short. This Gadget does not make any attempt to scale the data, it is assumed to be scaled upon entry.

-   [gpuRadialSensePrepGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_radial_sense_prep_gadget.html) (`gadgetron_radial`):

    This gadget is a SENSE preparation gadget for radial trajectories. It assembles a Sense job to be used in a subsequent solver. This Sense job contains all the arrays required for a subsequent reconstruction; coil sensitivity maps, a regularization image, sampling trajectory, data samples, and density compensation weights.
    This gadget has a number of configuration options:
    • mode. Please see the explanation in the header file [gpuRadialSensePrepGadget.h](https://gadgetron.github.io/api_master//gpu_radial_sense_prep_gadget_8h_source.html). Briefly: mode 0 and mode 1 denote fixed angle trajectories and mode 2 is based on the golden ratio.
    • deviceno. The gpu on which to run.
    • profiles_per_frame. The number of profiles (radial readouts) per reconstructed image. Must be specified for golden ratio based acquisitions but is normally auto-detected for modes 0-1. More details available in the header [gpuRadialSensePrepGadget.h](https://gadgetron.github.io/api_master//gpu_radial_sense_prep_gadget_8h_source.html).
    • frames_per_rotation. The number of image frames making up a fully sampled k-space (one rotation). For modes 0-1 the trajectory repeats itself after this many frames. Normally this parameter is auto-detected and for mode 2 (in which the name is less meaningful) it is set to 1 internally. Again, more details are available in the header [gpuRadialSensePrepGadget.h](https://gadgetron.github.io/api_master//gpu_radial_sense_prep_gadget_8h_source.html).
    • rotations_per_reconstruction. The number of rotations (see ‘frames_per_rotation’ above) to reconstruct per solver invocation. Can be set to 0 to denote a frame-by-frame reconstruction (the default for real-time reconstruction). The number of frames coming out of each solver invocation is then max(1,frames_per_rotation*rotations_per_reconstruction). Performance-wise it is often faster to reconstruct multiple frames (i.e. rotations) per reconstruction.
    • reconstruction_os_factor_x. The reconstruction matrix size is automatically determined from the input data, but this property denotes an internal oversampling factor (for an increased field of view) used during the reconstruction. After the reconstruction downstream the images are automatically cropped. Consider using this option if the radial profiles are oversampled in the readout direction.
    • reconstruction_os_factor_y. As above.
    • buffer_frames_per_rotation. Controls the size of the k-space buffer used to estimate coil sensitivity maps and the regularization image. Every time data for a new frame has been acquired, it is convolved into a k-space in accumulation mode. This property determines how many frames should be accumulated before estimating coil images. This is the innermost layer of the accumulation buffer. This option is automatically set internally if not specified.
    • buffer_length_in_rotations. This is the outermost layer of the accumulation buffer. This property defines how many of the most recent innermost buffers should be kept and summed to constitute the csm/regularization estimates.
    • buffer_using_solver. Set this property to ‘true’ if it is required that the regularization image has identical intensity levels as the reconstructed images. This is a requirement e.g. for prior image constrained compressed sensing.
    • buffer_convolution_kernel_width. Convolution kernel width for the accumulation buffer.
    • buffer_convolution_oversampling_factor. Oversampling factor used for the accumulation buffer convolutions.
    • sliding_window_profiles. Specifies how many profiles that are shared between subsequent frames. Setting this property (>0) increases the number of frames that are reconstructed. This property is (probably) only useful in mode 2 (golden ratio).
    • sliding_window_rotations. Specifies how many “rotations of profiles” that are shared between reconstructions. Setting this property (>0) causes some rotations (see ‘rotations_per_reconstruction’ above) to be reconstructed several times. This is useful if e.g. the quality of the first/latter rotation of a reconstruction is degraded; a rotation can then be discarded from the reconstruction in which it is an extreme rotation and kept in the reconstruction in which it is not. See e.g. the default kt-Sense config files. See also the property ‘rotations_to_discard’ for the solver gadgets below.
    • output_timing. Enable this property to see receive some timing feedback.

-   [gpuGenericSensePrepGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_generic_sense_prep_gadget.html) (`gadgetron_sense`):

    This gadget is a generic SENSE preparation gadget for arbitrary trajectories. It assembles a Sense job to be used in a subsequent solver. This Sense job contains all the arrays required for a subsequent reconstruction; coil sensitivity maps, a regularization image, sampling trajectory, data samples, and density compensation weights. It has a number of configuration options as described for the radial specialization above.

-   [gpuCgSenseGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_cg_sense_gadget.html) (`gadgetron_sense`):

    The gadget performs a linear sense reconstruction from a Sense job. It uses a linear conjugate gradient solver. A number of configuration options are available:

    • pass_on_undesired_data. A property that can be applied to all gadgets. If the incoming data is not of the expected type, it will be passed along downstream without generating an error. It is crucial to set this property if multiple gadgets for different sets/slices are used (see ‘setno’/’sliceno’ below). In such configurations the solver gadgets (except from the first solver gadget in the chain) will receive both the input data they desire but also the reconstructed images from the prior solver gadgets.
    • deviceno. The gpu on which to run.
    • setno. Solver settings for the data belonging to the set specified (see the ISMRMRD format description for further information). Multiple sets can be reconstructed in parallel and on different gpus.
    • sliceno. Solver settings for the data belonging to the slice specified (see the ISMRMRD format description for further information). Multiple slices can be reconstructed in parallel and on different gpus.
    • number_of_iterations. Number of iterations to run in the conjugate gradient solver (if the cg_limit is not reached).
    • cg_limit. Threshold of the relative residual before termination of the cg iterations.
    • oversampling_factor. Oversampling factor for the NFFT convolution.
    • kernel_width. Kernel width for the NFFT convolution.
    • kappa. Regularization weight.
    • rotations_to_discard. The number of rotations (see ‘rotations_per_frame’ for the prep gadget) to discard from a reconstruction. Should be an even number. Half of the specified rotations are discarded from the first frames of the reconstruction, the other half at the end. See also ‘sliding_window_rotations’ in the prep gadget.
    • output_convergence. Output solver convergence information in the Gadgetron terminal.
    • output_timing. Output solver performance timing information in the Gadgetron terminal.

-   [gpuCgKtSenseGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_cg_kt_sense_gadget.html) (`gadgetron_sense`):

    This gadget performs a linear kt-sense reconstruction from a Sense job. It uses a linear conjugate gradient solver. It contains all of the property options of the gpuCgSenseGadget and the following additional property.
    • training_data_shutter_radius. The radius of the circle around the center of k-space used for training data estimation. If this property is not specified it will be estimated automatically.

-   [gpuSbSenseGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1gpu_sb_sense_gadget.html) (`gadgetron_sense`):

    This gadget performs a non-linear sense reconstruction from a Sense job. It uses a constraint Split Bregman solver with total variation minimization - possibly with prior image regularization. Contains all of the property options of the gpuCgSenseGadget (except ‘number_of_iterations’ and ‘kappa’) and the following additional properties.
    • number_of_sb_iterations. The number of Split-Bregman iterations to perform (outer solver iterations).
    • number_of_cg_iterations. The number of conjugate gradient iterations for each Split-Bregman iteration (inner solver iterations).
    • mu. Regularization weight for the encoding operator.
    • lambda. Regularization weight of the regularization operators (partial derivative operators for total variation (TV) based regularization).
    • alpha. Weighing of the “pure” TV regularization term vs. the prior image term. The value should be between 0 and 1, where alpha=0 denotes TV regularization only and alpha=1 denotes prior regularization only.


-   [GrappaGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_grappa_gadget.html), [GrappaUnmixingGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_grappa_unmixing_gadget.html) (`gadgetron_grappa`):

    These Gadgets are used together to perform 2D Cartesian parallel imaging on the GPU. The [GrappaGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_grappa_gadget.html) is responsible for calculating GRAPPA coefficients and the [GrappaUnmixingGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_grappa_unmixing_gadget.html) Fourier transforms the raw data and applies the coefficients. The [GrappaGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_grappa_gadget.html) has the ability to use target channel compression, i.e. it can reconstruct using fewer target channels than input channels to improve performance. See [Gadgetron Applications] for details. The target channel compression is specificied like this:

        <gadget>
         <name>Grappa</name>
         <dll>gadgetrongrappa</dll>
         <class>GrappaGadget</class>
         <property><name>target_coils</name><value>8</value></property>
        </gadget>

-   [ImageFinishGadgetUSHORT](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget_u_s_h_o_r_t.html), [ImageFinishFLOAT](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget_f_l_o_a_t.html), [ImageFinishCPLX](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget_c_p_l_x.html)  (`gadgetron_mricore`):

    These 3 Gadgets are all template instances of the same [ImageFinishGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_image_finish_gadget.html). The only different between them is that they operate on different types of image data types as indicated by their names. Their purpose is to return the reconstructed images to the output queue of the Gadgetron so that they can be returned to the client.

-   [NoiseAdjustGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget.html) (`gadgetron_mricore`):

    The Gadgetron has two noise pre-whitening Gadgets with similar names [NoiseAdjustGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget.html) and [NoiseAdjustGadget_unoptimized](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget__unoptimized.html). They both perform the same operation, which is a) to collect noise adjust data when present, calculate the noise decorrelation matrix, and perform noise decorrelation (when the noise adjustment data is available). The difference between the two Gadgets is that [NoiseAdjustGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget.html) uses BLAS and LAPACK routines to perform the operation, which makes it much faster than unoptimized version. The latter Gadget is provided to enable reconstruction on systems where those libraries are not available.

-   [NoiseAdjustGadget_unoptimized](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget__unoptimized.html) (`gadgetron_mricore`):

    See description of [NoiseAdjustGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_noise_adjust_gadget.html).

-   [PCACoilGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_p_c_a_coil_gadget.html) (`gadgetron_mricore`):

    This Gadget is used to create virtual channels based on principal component analysis of a portion of the data. Specifically, data is accumulated for the first frame (for each location, i.e. slice) and a principal component analysis is done of this data. Once the PCA coefficients are available, all subsequent data will be transformed into the virtual channel domain and passed on down the Gadget chain. This Gadget is often combined with the [CoilReductionGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_coil_reduction_gadget.html).

-   [RemoveROOversamplingGadget](https://gadgetron.github.io/api_master//class_gadgetron_1_1_remove_r_o_oversampling_gadget.html) (`gadgetron_core`):

    Removes the 2x oversampling often used in the readout direction for (Cartesian) MRI.

### Python Gadgets

The Gadgetron provides a mechanism to do prototype development in Python. Again, we use MRI as the example application.

The Python layer is accessed through a set of Python Gadgets that can encapsulate a Python module. This is seen in the figure below, which illustrates a part of a Gadget chain with two Python Gadgets and one C/C++ Gadget. A Gadget chain can have any number of Python Gadgets and Python Gadgets can be mixed with C++ Gadgets.

<img src="http://gadgetron.sf.net/figs/python.png" style="width: 400px;" />

The Python modules that are encapsulated in the Python Gadgets are expected to have certain characteristics. Specifically, the Gadgets must have at least 3 functions and these functions will be called by the Gadgetron framework at certain specific times:

1.  *Gadget reference function*. A specific function will be called when the Python Gadget is created. This function is expected to receive a GadgetReference which is a class (wrapped in a Python module), which holds a reference to the Gadget, which owns the Python module. The purpose of passing this reference is to allow the Python module to return data to the Gadget when reconstruction outputs are ready. See below for details.

2.  *Configuration function*. This function is used to receive the configuration (usually in XML format), when it is passed to the Gadget, i.e. it is the Python equivalent of `process_config` in the Gadget (see ?).

3.  *Reconstruction function*. This function is called when the Gadget receives data, i.e. it is the Python equivalent of the `process` function in the Gadget (see [Gadgetron Streaming Architecture]).

The user can chose the names of these functions freely in the Python module, but the function names must be specified when the Gadget is inserted in the XML configuration:

    <gadget>
     <name>AccReconPython</name>
     <dll>gadgetronpython</dll>
     <class>AcquisitionPythonGadget</class>

     <property>
      <name>python_path</name>
      <value>/home/myuser/scripts/python</value>
     </property>

     <property>
      <name>python_module</name>
      <value>accumulate_and_recon</value>
     </property>

     <property>
      <name>gadget_reference_function</name>
      <value>set_gadget_reference</value>
     </property>

     <property>
      <name>input_function</name>
      <value>recon_function</value>
     </property>

     <property>
      <name>config_function</name>
      <value>config_function</value>
     </property>
    </gadget>

Notice how the 3 function names are specified through the `gadget_reference_function`, `input_function`, and `config_function` parameter names. Also notice that it is possible to specify a `python_path` to let the Python interpreter know where to search for script. By default, the `gadgetron/lib` is added to the search path. Multiple pathnames can be added by separating the paths with "`;`".

The Python script referenced in the XML configuration above could look like this:

    import numpy as np
    import GadgetronPythonMRI as g
    import kspaceandimage as ki
    import libxml2

    myLocalGadgetReference = g.GadgetReference()
    myBuffer = 0
    myParameters = 0
    myCounter = 1;
    mySeries = 1;

    def set_gadget_reference(gadref):
        global myLocalGadgetReference
        myLocalGadgetReference = gadref

    def config_function(conf):
        global myBuffer
        global myParameters

        myParameters = dict()

        doc = libxml2.parseDoc(str(conf))
        context = doc.xpathNewContext()
        context.xpathRegisterNs("ismrm", "http://www.ismrm.org/ISMRMRD")
        myParameters["matrix_x"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:x")[0]).content)
        myParameters["matrix_y"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:y")[0]).content)
        myParameters["matrix_z"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodedSpace/ismrm:matrixSize/ismrm:z")[0]).content)
        myParameters["channels"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:acquisitionSystemInformation/ismrm:receiverChannels")[0]).content)
        myParameters["slices"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodingLimits/ismrm:slice/ismrm:maximum")[0]).content)+1
        myParameters["center_line"] = int((context.xpathEval("/ismrm:ismrmrdHeader/ismrm:encoding/ismrm:encodingLimits/ismrm:kspace_encoding_step_1/ismrm:center")[0]).content)

        myBuffer = (np.zeros((myParameters["channels"],myParameters["slices"],myParameters["matrix_z"],myParameters["matrix_y"],(myParameters["matrix_x"]>>1)))).astype('complex64')

    def recon_function(acq, data):
        global myLocalGadgetReference
        global myBuffer
        global myParameters
        global myCounter
        global mySeries

        line_offset = (myParameters["matrix_y"]>>1)-myParameters["center_line"];
        myBuffer[:,acq.idx.slice,acq.idx.kspace_encode_step_2,acq.idx.kspace_encode_step_1+line_offset,:] = data
        
        if (acq.flags & (1<<7)): #Is this the last scan in slice
            image = ki.ktoi(myBuffer,(2,3,4))
            image = image * np.product(image.shape)*100 #Scaling for the scanner
            #Create a new image header and transfer value
            img_head = g.ImageHeader()
            img_head.channels = acq.active_channels
            img_head.slice = acq.idx.slice
            g.img_set_matrix_size(img_head, 0, myBuffer.shape[4])
            g.img_set_matrix_size(img_head, 1, myBuffer.shape[3])
            g.img_set_matrix_size(img_head, 2, myBuffer.shape[2])
            g.img_set_position(img_head, 0,g.acq_get_position(acq,0))
            g.img_set_position(img_head, 1,g.acq_get_position(acq,1))
            g.img_set_position(img_head, 2,g.acq_get_position(acq,2))
            g.img_set_read_dir(img_head, 0,g.acq_get_read_dir(acq,0))
            g.img_set_read_dir(img_head, 1,g.acq_get_read_dir(acq,1))
            g.img_set_read_dir(img_head, 2,g.acq_get_read_dir(acq,2))
            g.img_set_phase_dir(img_head, 0,g.acq_get_phase_dir(acq,0))
            g.img_set_phase_dir(img_head, 1,g.acq_get_phase_dir(acq,1))
            g.img_set_phase_dir(img_head, 2,g.acq_get_phase_dir(acq,2))
            g.img_set_slice_dir(img_head, 0,g.acq_get_slice_dir(acq,0))
            g.img_set_slice_dir(img_head, 1,g.acq_get_slice_dir(acq,1))
            g.img_set_slice_dir(img_head, 2,g.acq_get_slice_dir(acq,2))
            g.img_set_patient_table_position(img_head, 0, g.acq_get_patient_table_position(acq,0))
            g.img_set_patient_table_position(img_head, 1, g.acq_get_patient_table_position(acq,1))
            g.img_set_patient_table_position(img_head, 2, g.acq_get_patient_table_position(acq,2))
            img_head.acquisition_time_stamp = acq.acquisition_time_stamp
            img_head.image_index = myCounter;
            img_head.image_series_index = mySeries;

            myCounter = myCounter + 1
            if (myCounter > 5):
                mySeries = mySeries + 1
                myCounter = 1

            #Return image to Gadgetron
            return myLocalGadgetReference.return_image(img_head,image.astype('complex64'))

            #print "Returning to Gadgetron"
            return 0 #Everything OK

There is a lot going on in this script. Let us walk through the different parts and add some explanation. First look at the imports:

    import numpy as np
    import GadgetronPythonMRI as g
    import GadgetronXML
    import kspaceandimage as ki

All the Python Gadget modules must include `numpy`. The arrays (NDArray) are passed to the Python module as `numpy` arrays. The second module `GadgetronPythonMRI` is a Python wrapped version of some of the data
structures used in the MRI part of the Gadgetron. Specifically, the IMRMRD::AcquisitionHeader and ISMRMRD::ImageHeader headers are wrapped as Python types (using Boost Python). The `GadgetronPythonMRI` also contains a wrapped version of the GadgetReference class:

    class GadgetReference
    {

     public:
      GadgetReference();
      ~GadgetReference();
      
      int set_gadget(Gadget* g)
      {
        gadget_ = g;
        return 0;
      }

      template<class T> int return_data(T header, 
              boost::python::numeric::array arr);

      int return_acquisition(ISMRMRD::AcquisitionHeader acq, 
              boost::python::numeric::array arr);

      int return_image(ISMRMRD::ImageHeader img, 
              boost::python::numeric::array arr);

     protected:
      Gadget* gadget_;

    };

Using the return functions in this class interface, it is possible for the Python module to return data to the Gadget. `GadgetronXML` is a Python module provided with the Gadgetron, which contains some XML
helper functions that can (it is not a requirement) be used to parse the XML parameters that the module will receive from `process_config`. `kspaceandimage` is also a python module provided with the Gadgetron, it contains some simple wrapper functions for performing Fourier transforms (to and from k-space) of MRI data. The following section contains some initialization of global variables in the Python module;

    myRef = g.GadgetReference()
    myBuffer = 0
    myParameters = 0
    myCounter = 1;
    mySeries = 1;

As described above, each Python module must contain at least 3 functions corresponding to the 3 entry points from the Gadgetron framework. The first one of these functions captures the GadgetReference:

    def set_gadget_reference(gadref):
        global myLocalGadgetReference
        myLocalGadgetReference = gadref

Using this reference, the Python module will be able to return images (or acquisitions) to the Gadget. The next function (`config_function` processes the configuration data and finally, the `recon_function`
simply takes the data as it comes it and stores it in a buffer. Based on the `flags` field in the header, it is determined when the last acquisition in each slice has arrived. As this happens the buffer is Fourier transformed, an image header is populated, and the result is returned (via the GadgetReference) to the Gadgetron where it will be processed by the next Gadget in the chain.

The Gadgetron distribution comes with a simple Python-based 2D FT MRI reconstruction. The Gadget chain configuration for this reconstruction can be found in `gadgets/python/python.xml`.
This example shows how to use the forwards and adjoint non-Cartesian Fast Fourier Transform (NFFT and NFFT<sup>H</sup> respectively) on a 2D image. The source code can be found at
\$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/nfft/2d/main_nfft.cpp`
and \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/nfft/2d/main_nffth.cpp`.

We reuse the Shepp-Logan data downloaded for the previous experiments ([Image Denoising]).

In the following we run the NFFT followed by the NFFT<sup>H</sup>. The image matrix size is 256<sup>2</sup>. We use an oversampled matrix size of 384<sup>2</sup>, 128 profiles in k-space (*undersampling*) with 384 samples each. The NFFT Kaiser-Bessel convolution kernel width is set to 5.5<sup>2</sup> (see [Gadgetron Toolboxes]).

    user@host$ nfft -d shepp_logan_256_256_no_noise.real -o 384 -p 128 -s 384 -k 5.5
     Running reconstruction with the following parameters: 
    ---------------------------------------------------- 
      Input image file name (.real)  : shepp_logan_256_256_no_noise.real 
      Result file name               : samples.cplx 
      Oversampled matrix size        : 384 
      Number of profiles             : 128 
      Samples per profiles           : 384 
      Kernel width                   : 5.5 
    ---------------------------------------------------- 
    Loading image from disk
    Uploading, normalizing and converting to complex
    Initializing plan
    Computing golden ratio radial trajectories
    NFFT preprocessing
    Computing density compensation weights
    Computing nfft (inverse gridding)
    Output result to disk
    user@host$

    user@host$ nffth -d samples.cplx -m 256 -o 384 -k 5.5
     Running reconstruction with the following parameters: 
    ---------------------------------------------------- 
      Input samples file name (.cplx)  : samples.cplx 
      Output image file name (.cplx)   : result.cplx 
      Matrix size                      : 256 
      Oversampled matrix size          : 384 
      Kernel width                     : 5.5 
    ---------------------------------------------------- 
    Loading samples from disk
    Uploading samples to device
    Initializing plan
    Computing golden ratio radial trajectories
    NFFT preprocessing
    Computing density compensation weights
    Computing nffth (gridding)
    Output result to disk
    user@host$

The result is saved in file `result.cplx`. A magnitudes image is saved as `result.real`. As an exercise, experiment with the settings to reduce (or increase) the aliasing.

The \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/MRI/nfft/2d/` folder also contains examples of using the nfftOperator in a Conjugate Gradient solver and a Split Bregman solver respectively.

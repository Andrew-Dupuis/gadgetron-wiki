This example uses 1) the linear least squares solver, and 2) the constraint Split Bregman solver for image deblurring. The encoding matrix is defined as a convolutionOperator. A partialDerivativeOperator is added for each of the two spatial directions as regularization terms.

We reuse the Shepp-Logan data from the image denoising experiment above ([Image Denoising]).

First we generate a blurry Shepp-Logan phantom by convolution with a Gaussian kernel. This is easily achieved using the method `mult_M` in the convolutionOperator. Source code is provided at \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/deblurring/2d/blur_2d.cpp`

In a terminal, go to the folder in which you unpacked the Shepp-Logan phantom.

Try

    user@host$ blur_2d -d shepp_logan_256_256_no_noise.real

which generates two complex images; `blurred_image.cplx` and `kernel_image.cplx`. For convenience a corresponding magnitudes image is also saved as `blurred_image.real`.

Next run the conjugate gradient solver. The source code for the example
can be found in \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/deblurring/2d/deblur_2d_cg.cpp`.

    user@host$ deblur_2d_cg -K 1e-4
     Running deblurring with the following parameters: 
    ---------------------------------------------------- 
      Blurred image file name (.cplx)  : blurred_image.cplx 
      Kernel image file name (.cplx)   : kernel_image.cplx 
      Result file name                 : cg_deblurred_image.cplx 
      Number of iterations             : 25 
      Regularization weight            : 1e-4 
    ---------------------------------------------------- 
    Iterating...
    ...
    user@host$

The result is saved in the current folder in the file `cg_deblurred_image.cplx`. A magnitudes image is also saved as `cg_deblurred_image.real`.

Next run the constraint Split Bregman solver. The source code for the example can be found in \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/deblurring/2d/deblur_2d_sb.cpp`.

    user@host$ deblur_2d_sb -O 100 -L 0.5 -M 0.5
     Running deblurring with the following parameters: 
    ---------------------------------------------------- 
      Blurred image file name (.cplx)  : blurred_image.cplx 
      Kernel image file name (.cplx)   : kernel_image.cplx 
      Result file name                 : sb_deblurred_image.cplx 
      Number of cg iterations          : 20 
      Number of sb inner iterations    : 1 
      Number of sb outer iterations    : 100 
      Mu                               : 0.5 
      Lambda                           : 0.5 
    ---------------------------------------------------- 
    ...
    user@host$

The result is saved as `sb_deblurred_image.cplx`. A magnitudes image is also saved as `sb_deblurred_image.real`.

The blurred and deblurred phantoms are depicted below.

Blurred phantom:

<img src="http://gadgetron.sf.net/figs/shepp_blurred.png" style="width: 400px;" />

Deblurred phantom from the Conjugate Gradient:

<img src="http://gadgetron.sf.net/figs/shepp_deblurred_cg.png" style="width: 400px;" />

Deblurred phantom from the Conjugate Gradient solver:
<img src="http://gadgetron.sf.net/figs/shepp_deblurred_sb.png" style="width: 400px;" />

In the present examples no noise was added to the blurred images before the deconvolution. Consequently for the conjugate gradient solver, a very low weight of the regularization term was "sufficient". We leave it as an exercise to run the algorithms with various settings. In particular, try to add noise to the blurred image before the deconvolution to observe the very ill-posed nature of the problem.

Notice. If the dimensions of the provided convolution kernel is exactly double that of the provided image, the convolution operator zero-pads the image before the convolution and removes the padding again after. As the convolution operator utilizes FFTs in its implementation, this oversampling is a way of avoiding cyclic boundary conditions during the convolution.
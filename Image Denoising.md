This example uses the unconstraint Split Bregman solver for total variation based 2D image denoising. The encoding matrix is defined as an identityOperator and a partialDerivativeOperator is used for each of the two spatial directions to implement the total variation regularization term. The two partial derivatives are added as a "group" of regularization operators to implement isotropic denoising. Alternatively, by changing a few lines of code they can be added as individual regularization operators instead to implement anisotropic denoising.

The full source code for the example can be found at \$(GADGETRON\_SOURCE)`/apps/standalone/gpu/denoising/2d/denoise_TV.cpp`.

You can download some noisy Shepp-Logan phantom test datasets from <https://gadgetrondata.blob.core.windows.net/gadgetrongithubio/files/testdata/standalone/shepp.tar.gz>

In a terminal, go to the folder in which you unpacked the data. We assume that you have added \$(GADGETRON\_HOME)/bin to your PATH environment variable.

Try

    user@host$ denoise_TV -d shepp_logan_256_256_med_noise.real -O 250 -m 1
    Running denoising with the following parameters: 
    ---------------------------------------------------- 
      Noisy image file name (.real)  : shepp_logan_256_256_med_noise.real 
      Result file name               : denoised_image_TV.real 
      Number of cg iterations        : 20 
      Number of sb inner iterations  : 1 
      Number of sb outer iterations  : 250 
      Regularization weight (mu)     : 1 
    ---------------------------------------------------- 
    ...
    user@host$

which runs 250 iterations of the solver with a regularization weight of 1. The output is saved in the current folder in the file `denoised_image_TV.real`.

The noisy and denoised phantom is depicted below.

<img src="http://gadgetron.sf.net/figs/shepp_noisy.png" style="width: 400px" />

<img src="http://gadgetron.sf.net/figs/shepp_denoised.png" style="width: 400px" />

Running denoise\_TV with no arguments prints out a brief usage description. We leave it as an exercise to run the algorithm with various settings. The data file you downloaded contains two further dataset (with lower and higher noise levels respectively) to try out as well.

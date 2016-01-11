# Installation in more detail

This is intended as a more explanatory guide to Gadgetron installation for those less familiar with installation processes. Some of it is Mac specific. In brief, Gadgetron and ISMRMRD source code and data that you download goes into directories below your home directory, you then compile this code (using `make`) and the executables get installed in directories below `/usr/local` when you type `make install`. Generally, the system reserves `/usr/bin` etc for its use, and developers install into `/usr/local/`. 
Code looks for executables in preset folders, and other places that you can specify using PATH-type environment variables or flags to compilers. Extensions to PATH-type variables are usually placed in a user's `.bash_profile` file in their home directory. Note that just editing this file has no effect, the changes only take place if you force them, or for each new window opened after you save the file. 

Python is also required and be aware that there are multiple versions, not just 2.7 vs 3.3, 3.4 or 3.5, but also of 2.7 itself. Applications can install their own Python (e.g. anaconda) and users can download versions. Unfortunately these versions are not all compatible and this can lead to problems. 


## Useful commands
`ps -ax | grep gadgetron` to see if there are any gadgetron processes running on your machine. You can use the `kill` command to stop them if necessary.

Note the `/usr` folder does not show automatically in a Mac Finder.
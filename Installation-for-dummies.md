# Installation in more detail

This is intended as a more explanatory guide to Gadgetron installation for those less familiar with installation processes. Some of it is Mac specific. In brief, Gadgetron and ISMRMRD source code and data that you download goes into directories below your home directory, you then compile this code (using `make`) and the executables get installed in directories below `/usr/local` when you type `make install`. Generally, the system reserves `/usr/bin` etc for its use, and user's complied code gets installed into directories below `/usr/local/`. 

Executing code and scripts often look for executables in preset folders, and other places that you can specify using PATH-type environment variables or flags to compilers. Extensions to PATH-type variables are usually placed in a user's `.bash_profile` file in their home directory. Note that just editing this file has no effect, the changes only take place if you force them, or for each new window opened after you save the file. 

Python is also required and be aware that there are multiple versions, not just 2.7 vs 3.3, 3.4 or 3.5, but also of 2.7 itself. Applications can install their own Python (e.g. anaconda) and users can download versions. Unfortunately these versions are not all compatible and this can lead to problems. For the Mac OS X installation, I would recommend having just the Apple supplied version, and that installed by Homebrew.

The Mac OS X installation uses Homebrew to install a lot of the libraries etc required by Gadgetron and ISMRMRD. Homebrew puts stuff in directories below `/usr/local/Cellar` and then provides symbolic links to this folder. Homebrew makes a good try at keeping things in order. A very useful command is `brew doctor` that can find problems. Note it will question if gadgetron, ismrmrd and gtest libraries should be present and you have installed them - yes they should be there.

## Installation Process
Follow the instructions in the order described in the Wiki. Note that if you have any OS upgrades to be made, do them  **before** starting this installation (and for major upgrades, you may find things have not been tested).


## Useful commands
`ps -ax | grep gadgetron` to see if there are any gadgetron processes running on your machine. You can use the `kill` command to stop them if necessary.

Note the `/usr` folder does not show automatically in a Mac Finder.
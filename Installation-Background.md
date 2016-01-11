# Installation in more detail

This is intended as a more explanatory guide to Gadgetron installation for those less familiar with installation processes. Some of this is Mac specific. In brief, Gadgetron and ISMRMRD source code and data that you download goes into directories below your home directory (e.g. into `~/gadgetron/`), you then compile this code (using `make`) and when you type `make install` the executables get installed in directories below `/usr/local`. Generally, the system reserves `/usr/bin` etc for its use, and user's compiled code gets installed into directories below `/usr/local/`. The Gadgetron test scripts and data will be based below your home in the directory `~/gadgetron/test/integration`. 

Executing code and scripts look for executables in preset folders, and other places that are specified using PATH-type environment variables or flags to compilers. Extensions to PATH-type variables are usually placed in a user's `.bash_profile` file in their home directory. Note that just editing this file has no immediate effect, the changes only take place if you force them, or for each new terminal window opened after you save the file. 

Python is required and be aware that there are multiple versions, not just 2.7 vs 3.3, 3.4 or 3.5, but also of 2.7 itself. External applications can install their own Python (e.g. anaconda) and users can download versions. Unfortunately these versions are not all compatible and this can lead to problems. For the Mac OS X installation, I would recommend having just the Apple supplied version, and that installed by Homebrew.

The Mac OS X installation uses Homebrew to install a lot of the libraries etc required by Gadgetron and ISMRMRD. Homebrew puts stuff in directories below `/usr/local/Cellar` and then provides symbolic links to this folder. Homebrew makes a good try at keeping things in order. A very useful command is `brew doctor` that can find problems. Note it will question if gadgetron, ismrmrd and gtest libraries should be present and after you have installed them - yes they should be there.

## Installation Process
If you have any OS upgrades to be made, do them  before starting this installation (and for major OS upgrades, you may decide to wait some time after release before upgrading to give others a chance to test features). Follow the Wiki instructions **in order**. On a Mac, if you have previously installed Python from their main site, or as part of anaconda, you may find that your `~/.bash_profile` file has had extra lines added at the end modifying the Python path. You should comment these lines out. I have also found that on an El Capitan Mac, I need the following two lines added to this file:
```
export PYTHONPATH=$PYTHONHOME/lib/python2.7/site-packages
export PYTHONPATH=/usr/local/opt/libxml2/lib/python2.7/site-packages:$PYTHONPATH
```
Remember that changes to `.bash_profile` do not take effect in old terminal windows unless you force them (so open new windows). After installing Homebrew, check for updates and the `brew doctor` before continuing. During the `brew install` processes, there will be a lot of messages. Keep an eye on these. In particular if it reports any directories cannot be written to, run `brew doctor` and follow advice about how to change ownership.

If you need to repeat a step, then you probably need to repeat all steps after that to ensure the code and executables are all consistent. If repeating a `make` step, I find it safest to delete the corresponding build folder (using `rm -rf build`) before using `mkdir build` to create a new, clean version. 



## Useful commands
`ps -ax | grep gadgetron` to see if there are any gadgetron processes running on your machine. You can use the `kill` command to stop them if necessary.

Note the `/usr` folder does not show automatically in a Mac Finder.
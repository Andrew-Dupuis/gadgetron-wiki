Docker (http://docker.com) is a convenient to deploy complete Gadgetron installations (including all dependencies). It is similar to a chroot approach where a complete Linux file system (an image) is deployed on a Linux host. Once deployed, it is an isolated environment with separate filesystem and namespaces for processes, etc. Please refer to the Docker documentation at https://docs.docker.com/ for more details.

We provide Docker images of the Gadgetron, which can be found at Docker Hub (https://hub.docker.com/r/hansenms/gadgetron/). Using these images it is possible to quickly download and deploy a Gadgetron installation. 

Installing the Docker host components is easy on most Linux machines:

    $ curl -sSL https://get.docker.com/ | sh

Please refer to the Docker documentation for your specific flavor of Linux. It is also possible to install a VM that runs a Docker host on Mac OS X and Windows. If you chose this route, you will be able to run a Gadgetron Docker container on your Windows or Mac, but be aware that it will run in a VM, so performance will not be that of a native Linux installation. 
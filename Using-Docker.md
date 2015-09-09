Docker (http://docker.com) is a convenient to deploy complete Gadgetron installations (including all dependencies). It is similar to a chroot approach where a complete Linux file system (an image) is deployed on a Linux host. Once deployed, it is an isolated environment with separate filesystem and namespaces for processes, etc. Please refer to the Docker documentation at https://docs.docker.com/ for more details.

We provide Docker images of the Gadgetron, which can be found at Docker Hub (https://hub.docker.com/r/hansenms/gadgetron/). Using these images it is possible to quickly download and deploy a Gadgetron installation. 

Installing the Docker host components is easy on most Linux machines:

    $ curl -sSL https://get.docker.com/ | sh

Please refer to the Docker documentation for your specific flavor of Linux. 

It is also possible to install a VM that runs a Docker host on Mac OS X and Windows. If you chose this route, you will be able to run a Gadgetron Docker container on your Windows or Mac, but be aware that it will run in a VM, so performance will not be that of a native Linux installation. See the details on the Docker toolbox (https://www.docker.com/toolbox) for more information. 

Once you have installed Docker, you can download and start the Gadgetron with a command like:

    export CUDA_DEVICES="--device=/dev/nvidia0:/dev/nvidia0 --device=/dev/nvidiactl:/dev/nvidiactl --device=/dev/nvidia-uvm:/dev/nvidia-uvm"
    docker run ${CUDA_DEVICES} --name gt1 -p 9002:9002 -p 8090:8090 -p 8002:8002 --rm -t hansenms/gadgetron

The CUDA_DEVICES part is only needed if you have GPUs on your host system and you would like to use them in the container. Please type:

    ls -a /dev/* |grep nvidia

To see which CUDA devices you have available on your host. You should map them all into the Docker container. The mapping of CUDA devices is only relevant if a) you are running natively on Linux and b) you actually have NVIDIA GPUs. 

The Dockerfile configurations that are available can be found in https://github.com/gadgetron/gadgetron/tree/master/docker where we will be adding more configurations as we go. 

You can also build a docker images locally from one of the configurations:

    cd cuda_331_openblas/
    docker build --no-cache -t gadgetron .

The `--no-cache` option is optional. The reason for using it is to ensure that the `git clone` statements in the Dockerfile actually pull a fresh version of the source code. 
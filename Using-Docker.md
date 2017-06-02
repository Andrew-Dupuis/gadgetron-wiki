Docker (http://docker.com) is a convenient to deploy complete Gadgetron installations (including all dependencies). It is similar to a chroot approach where a complete Linux file system (an image) is deployed on a Linux host. Once deployed, it is an isolated environment with separate filesystem and namespaces for processes, etc. Please refer to the Docker documentation at https://docs.docker.com/ for more details.

We provide Docker images of the Gadgetron, which can be found at Docker Hub (https://hub.docker.com/r/gadgetron/). Using these images it is possible to quickly download and deploy a Gadgetron installation. There are 3 main images at the moment:

* `gadgetron/ubuntu_1604_cuda75` which is a complete Gadgetron installation compiled against CUDA 7.5
* `gadgetron/ubuntu_1604_cuda55` which is a complete Gadgetron installation compiled against CUDA 5.5
* `gadgetron/ubuntu_1604_no_cuda` which is a Gadgetron without CUDA support.

Installing the Docker host components is easy on most Linux machines. Please consult the Docker documentation at https://docs.docker.com/engine/installation/

It is also possible to install a VM that runs a Docker host on Mac OS X and Windows. If you chose this route, you will be able to run a Gadgetron Docker container on your Windows or Mac, but be aware that it will run in a VM, so performance will not be that of a native Linux installation. See the details on the Docker toolbox (https://www.docker.com/toolbox) for more information. 

Once you have installed Docker, you can download the Gadgetron with a command like:
   
    docker pull gadgetron/ubuntu_1604_cuda75

To start the image you need to either use the `docker run` command or the `nvidia-docker run` command, which you can get at https://github.com/NVIDIA/nvidia-docker. We have also included an older `nvidia-docker` script in `docker/nvidia-docker`. The purpose of the `nvidia-docker` is to provide a wrapper around the docker command so that your GPU devices get exposed inside the containers. With the `nvidia-docker` script that is included in the Gadgetron repo, you can start the container with something like:

    GPU=0 ${GADGETRON_SOURCE}docker/nvidia-docker run --name=gt1 --publish=9002:9002 --publish=8090:8090 --publish=8002:8002 --rm -t gadgetron/ubuntu_1604_cuda75

The `GPU=0` tells nvidia-docker to expose only GPU 0 in the container. To see which CUDA devices you have available on your host type `nvidia-smi`. If you are using the newer `nvidia-docker` mentioned above, the command would simply be:

    NV_GPU=0 nvidia-docker run --name=gt1 --publish=9002:9002 --publish=8090:8090 --publish=8002:8002 --rm -t gadgetron/ubuntu_1604_cuda75
In this example we are mapping the ports 8002 (cloudbus relay), 8090 (the web app monitor), and 9002 (the Gadgetron itself) through to the host system. You can check on the status of your Gadgetron in the container by pointing your browser to:

    http://<ADDRESS OF DOCKER HOST>:8090/gadgetron

And you can send data to the Gadgetron using port 9002 of your Docker host. Bear in mind that your Docker host may be the IP address of the VM if you are running it using the Docker Toolbox. 

The Dockerfile configurations that are available can be found in https://github.com/gadgetron/gadgetron/tree/master/docker where we will be adding more configurations as we go. 

You can also build a docker images locally from one of the configurations:

    cd ${GADGETRON_SOURCE}/docker/incremental/ubuntu_1404_cuda75/
    docker build --no-cache -t my_gadgetron_image .

The `--no-cache` option is optional. The reason for using it is to ensure that the `git clone` statements in the Dockerfile actually pull a fresh version of the source code. 

The `Dockerfile`s use a number of base images with all the required dependencies installed. They can be found on https://hub.docker.com/r/gadgetron and the `Dockerfile` are in `${GADGETRON_SOURCE}/docker/base`

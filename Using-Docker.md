Docker (http://docker.com) is a convenient to deploy complete Gadgetron installations (including all dependencies). It is similar to a chroot approach where a complete Linux file system (an image) is deployed on a Linux host. Once deployed, it is an isolated environment with separate filesystem and namespaces for processes, etc. Please refer to the Docker documentation at https://docs.docker.com/ for more details.

We provide Docker images of the Gadgetron, which can be found at Docker Hub (https://hub.docker.com/r/gadgetron/). Using these images it is possible to quickly download and deploy a Gadgetron installation. There are 3 main images at the moment:

* `gadgetron/ubuntu_1604_cuda80` which is a complete Gadgetron installation compiled against CUDA 8.0
* `gadgetron/ubuntu_1604_cuda80_cudnn6` which is a complete Gadgetron installation compiled against CUDA 8.0 and cudnn6. This version also had the tensorflow package installed, allowing the development of AI applications
* `gadgetron/ubuntu_1604_no_cuda` which is a Gadgetron without CUDA support.

Installing the Docker host components is easy on most Linux machines. Please consult the Docker documentation at https://docs.docker.com/engine/installation/

It is also possible to install a VM that runs a Docker host on Mac OS X and Windows. If you chose this route, you will be able to run a Gadgetron Docker container on your Windows or Mac, but be aware that it will run in a VM, so performance will not be that of a native Linux installation. See the details on the Docker toolbox (https://www.docker.com/toolbox) for more information. 

For ubuntu 16.04, following commands can be used to install docker-ce

```
sudo apt-get remove docker docker-engine docker.io
cd ~/software
wget https://download.docker.com/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_17.09.1~ce-0~ubuntu_amd64.deb
sudo dpkg -i docker-ce_17.09.1~ce-0~ubuntu_amd64.deb
sudo docker run hello-world
```

Once you have installed Docker, you can download the Gadgetron with a command like:
   
    docker pull gadgetron/ubuntu_1604_cuda80

To start the image you need to either use the `docker run` command or the `nvidia-docker` command, which you can get at https://github.com/NVIDIA/nvidia-docker. 

To install nvidia-docker, following the help at [nvidia-docker](https://github.com/NVIDIA/nvidia-docker/wiki/Installation-(version-2.0)).

For the ubuntu, follow commands install nvidia-docker 2:

```
# If you have nvidia-docker 1.0 installed: we need to remove it and all existing GPU containers
docker volume ls -q -f driver=nvidia-docker | xargs -r -I{} -n1 docker ps -q -a -f volume={} | xargs -r docker rm -f
sudo apt-get purge -y nvidia-docker

# Add the package repositories
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | \
  sudo apt-key add -
curl -s -L https://nvidia.github.io/nvidia-docker/ubuntu16.04/amd64/nvidia-docker.list | \
  sudo tee /etc/apt/sources.list.d/nvidia-docker.list
sudo apt-get update

# Install nvidia-docker2 and reload the Docker daemon configuration
sudo apt-get install -y nvidia-docker2
sudo pkill -SIGHUP dockerd
```
To start docker:

docker run --runtime=nvidia --name=gt1 --publish=9002:9002 --publish=9080:9080 --publish=8002:8002 --publish=9001:9001 --volume=/tmp/gadgetron_data:/tmp/gadgetron_data --rm -t gadgetron/ubuntu_1604_cuda80

nvidia-docker will expose all GPUs into the container. To see which CUDA devices you have available on your host type `nvidia-smi`. In this example we are mapping the ports 9001 (supervisord web interface), 8002 (cloudbus relay), 9080 (the web app monitor), and 9002 (the Gadgetron itself) through to the host system. You can check on the status of your Gadgetron in the container by pointing your browser to:

    http://<ADDRESS OF DOCKER HOST>:9001

And you can send data to the Gadgetron using port 9002 of your Docker host. Bear in mind that your Docker host may be the IP address of the VM if you are running it using the Docker Toolbox. 

The Dockerfile configurations that are available can be found in https://github.com/gadgetron/gadgetron/tree/master/docker where we will be adding more configurations as we go. 

You can also build a docker images locally from one of the configurations:

    cd ${GADGETRON_SOURCE}/docker/incremental/ubuntu_1604_cuda80/
    docker build --no-cache -t my_gadgetron_image .

The `--no-cache` option is optional. The reason for using it is to ensure that the `git clone` statements in the Dockerfile actually pull a fresh version of the source code. 

The `Dockerfile`s use a number of base images with all the required dependencies installed. They can be found on https://hub.docker.com/r/gadgetron and the `Dockerfile` are in `${GADGETRON_SOURCE}/docker/base`

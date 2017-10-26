We recommend [using Docker](https://github.com/gadgetron/gadgetron/wiki/Using-Docker) containers for deploying the Gadgetron. However, [chroot](https://en.wikipedia.org/wiki/Chroot) images are a possible convenient way of deploying the Gadgetron on a system that does not have [Docker](https://docker.com) installed. 

The recommended way to generate the chroot image is from the Docker container, for example to generate an image (both `*.tar.gz` and `*.img` (e.g. for deployment on Siemens scanner) use the following command:

```
    ${GADGETRON_SOURCE}/docker/create_chroot_from_image gadgetron/ubuntu1604_cuda80 6144
```

Since version 3.0, Gadgetron supports building the chroot image as a way for deployment on the Ubuntu system.

This will generate a couple of image files, e.g., `gadgetron-20171021-2323-51cd1ba4.img`, and `gadgetron-20171021-2323-51cd1ba4.tar`

The naming is gadgetron-date-time-first 8 numbers of SHA1 key for gadgetron code base.
The .img file is a hard-disk image file containing the same content with .tar file.

# Run gadgetron from chroot image

In the ${GADGETRON_SOURCE}/docker, there is a script called gchroot:

To exam the help information:
```
    sudo ${GADGETRON_SOURCE}/docker/gchroot

    Usage:  gchroot <COMMAND> [OPTIONS] <ARGS>

    Available commands:

       list                          : List mounted chroot images
       exec <mount point> <command>  : Execute command in chroot
       attach <img OR path>          : Prepare chroot (with mounts)
       detach <mount point>          : Remove chroot mount point
       run <img OR path> <command>   : Run command in image
       clean <mount point>           : Remove all processes accessing mount point

    Available options

      -r | --rm | --remove           : Detach chroot after run
      -n | --name <NAME>             : Use this name with attached chroot
      -v | --volume <SRC:DST>        : Mount SRC at DST in chroot
      --verbose                      : Verbose logging
```

# Mount the chroot image

```
    sudo .${GADGETRON_SOURCE}/docker/gchroot attach gadgetron-20171021-2323-51cd1ba4.img
```
# Find the mounting point

```
    sudo .${GADGETRON_SOURCE}/docker/gchroot list

    /mnt/gadgetron_chroot/chroot_9382_20171026_140306
```
# Run gadgetron from the chroot image

```
    sudo .${GADGETRON_SOURCE}/docker/gchroot run /mnt/gadgetron_chroot/chroot_9382_20171026_140306 gadgetron
```

```
10-26 14:03:50.747 INFO [main.cpp:200] Starting ReST interface on port 9080
10-26 14:03:50.760 INFO [main.cpp:212] Starting cloudBus: localhost:8002
10-26 14:03:50.761 INFO [main.cpp:260] Configuring services, Running on port 9002
```

Now a gadgetron is running at port 9002 of the computer.

# To remove the mounting point

```
    sudo .${GADGETRON_SOURCE}/docker/gchroot detach /mnt/gadgetron_chroot/chroot_9382_20171026_140306

    or

    sudo .${GADGETRON_SOURCE}/docker/gchroot detach all
```
# Using Matlab with Gadgetron in Docker
## Accessing the Host Matlab Installation within a Docker Container

Docker allows you to mount components from the host file system into a container at the time the containter is created and run (N.B. not, at the moment, after it has been created and run). We can use this facility to very painlessly access the host installation of Matlab from within a container. Let's look at a command for starting a gadgetron container to illustrate.

`NV_GPU=0 nvidia-docker run --name=gt1 --publish=9002:9002 --publish=8090:8090 --publish=8002:8002 -v /:/mnt -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY --rm -it gadgetron/ubuntu_1404_cuda75`

Everything is the same as normal for exposing the host cuda card and the gadgetron ports (see gadgetron wiki docker page) except for the

`-v /:/mnt -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY`

options

This first `-v /:/mnt` option mounts the entire _host_ file system to appear in the docker running image at at the /mnt mount point (which I think needs to exist in the client file system - or maybe will be automatically added / merged as part of the Docker layered file system running layer??). So, as shown below, for this example, it seems that all the (CentOS) host file systems are exported into the (Ubuntu) container. Dangerously cool for experimentation but probably better to have more specific exports for a production system!
```
[root@centoshost ~]# df
Filesystem               1K-blocks     Used  Available Use% Mounted on
/dev/mapper/centos-root   52403200 46721572    5681628  90% /
devtmpfs                  16312300        0   16312300   0% /dev
tmpfs                     16335808   150440   16185368   1% /dev/shm
tmpfs                     16335808   107900   16227908   1% /run
tmpfs                     16335808        0   16335808   0% /sys/fs/cgroup
/dev/sdb1               2928834748 64240232 2864594516   3% /scratch
/dev/sda6                   505580   341236     164344  68% /boot
/dev/sda2                   364544    68416     296128  19% /boot/efi
/dev/mapper/centos-home 2779703180 16215332 2763487848   1% /localhome
tmpfs                      3267164       16    3267148   1% /run/user/1234
tmpfs                      3267164        0    3267164   0% /run/user/0
[root@peaches ~]# docker exec -it gt1 bash
root@a445ae98b0b7:/# df
Filesystem                                                                                          1K-blocks     Used  Available Use% Mounted on
/dev/mapper/docker-253:1-68394805-e889429230e261d7c58845bbff1fe70d067cc4de85d46f3c53515ef29062faad   10474496  3603628    6870868  35% /
tmpfs                                                                                                16335808        0   16335808   0% /dev
tmpfs                                                                                                16335808        0   16335808   0% /sys/fs/cgroup
/dev/mapper/centos-root                                                                              52403200 46721452    5681748  90% /mnt
devtmpfs                                                                                             16312300        0   16312300   0% /mnt/dev
tmpfs                                                                                                16335808   150440   16185368   1% /mnt/dev/shm
tmpfs                                                                                                16335808        0   16335808   0% /mnt/sys/fs/cgroup
tmpfs                                                                                                16335808   107908   16227900   1% /mnt/run
tmpfs                                                                                                 3267164       16    3267148   1% /mnt/run/user/1234
tmpfs                                                                                                 3267164        0    3267164   0% /mnt/run/user/0
/dev/sdb1                                                                                          2928834748 64240232 2864594516   3% /mnt/scratch
/dev/sda6                                                                                              505580   341236     164344  68% /mnt/boot
/dev/sda2                                                                                              364544    68416     296128  19% /mnt/boot/efi
/dev/mapper/centos-home                                                                            2779703180 16215332 2763487848   1% /mnt/localhome
shm                                                                                                     65536        0      65536   0% /dev/shm
```

So, in particular, my host Matlab installations appear in the client docker image at
```
root@a445ae98b0b7:/# ls -l /mnt/usr/local/MATLAB
total 8
drwxr-xr-x. 21 root root 4096 Nov 19  2015 R2015b
drwxr-xr-x. 22 root root 4096 Mar 15 12:21 R2016a
```

## Using the host X server from within a Docker container to license and run Matlab

For some of our uses of Matlab with gadgetron we want to have access to the Matlab GUI or at least images, via X11. The installation and licence management is also simplified if X11 is available. There are several methods outlined in the following

http://wiki.ros.org/docker/Tutorials/GUI

but (for now...) I am using the quickest and dirtiest I could find

http://fabiorehm.com/blog/2014/09/11/running-gui-apps-with-docker/

`-v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY`

The gadgetron dockerhub image seemed also to need some x cruft installed. Getting an xterm to work seems to be enough.

`apt-get update && apt-get install xterm`

this does throw an error - which Google says may be to do with selinux on the host machine - although error persists when I setenforce 0??

`dpkg: error processing archive /var/cache/apt/archives/libelf1_0.158-0ubuntu5.2_amd64.deb (--unpack):
 cannot get security labeling handle: No such file or directory`

UPDATE: the this error does not occur, eg., on an Ubuntu 16.04 host.

So, then you can validate matlab against the gadgetron container HWAddr via the gui

`/mnt/usr/local/MATLAB/R2016a/bin/matlab`

and finally run the validated matlab (with the same command). Matlab will place a new `licence_XXXXX.lic` file in the host's `/usr/local/MATLAB/R2016a/licenses` direcory for the container's use of Matlab. I don't know, yet, what happens re licensing when the container is moved.

See also

https://github.com/RenderToolbox3/VirtualScenes/wiki/Matlab-on-Docker-and-EC2
***

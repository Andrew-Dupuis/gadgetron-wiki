In order to install Gadgetron using deb and pip packages, you need to update the /etc/apt/sources.list file:

Enable multiverse and restricted packages:

    deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty universe multiverse restricted
    deb-src http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty universe multiverse restricted
    deb http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty-updates universe multiverse restricted
    deb-src http://us-east-1.ec2.archive.ubuntu.com/ubuntu/ trusty-updates universe multiverse restricted

Add address of gadgetron PPA:

    deb http://ppa.launchpad.net/gadgetron/gadgetron/ubuntu trusty main
    deb-src http://ppa.launchpad.net/gadgetron/gadgetron/ubuntu trusty main


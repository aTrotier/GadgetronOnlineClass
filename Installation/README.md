# Installation Procedure

To install Gadgetron, it has traditionally been necessary to compile the source code. And while this is still an option (see below), we do reccomend that you take a look at the precompiled packages available through the Gradient Software PPA. This makes the Gadgetron software suite available on Ubuntu with very little hazzle.   

## Ubuntu

To install Gadgetron using the Gradient Software PPA, first add the PPA to your software sources: 
```bash 
sudo add-apt-repository ppa:gradient-software/experimental
sudo apt-get update
```

Gadgetron (and related software) can then be installed using the package manager:
```bash
sudo apt-get install gadgetron-all
```

It is also recommended that you install the Python interface:
```bash
pip3 install --user gadgetron
```

If you have Matlab, and intend to use it with Gadgetron, you can install the Gadgetron Matlab interface by searching for 'gadgetron' in the Add-Ons manager in Matlab. 

## Windows

While Gadgetron does compile natively on Windows, no binary package is available. We recommend that you in stead use WSL, and use the Ubuntu packages as described above.  

Microsoft has provided excellent documentation for [installing WSL](https://docs.microsoft.com/en-us/windows/wsl/about). 

Install Ubuntu 18.04 or 20.04, and follow the instructions above to complete your Gadgetron installation.

## Mac OS 

Gadgetron doesn't currently build on Mac OS.  You may try MacPorts or HomeBrew.  However, users should have better luck with the following options:

- Software such as Parallels, VirtualBox, etc., can be used to install a Ubuntu virtual machine (VM), then install Gadgetron into that Ubuntu VM, as detailed above for the native Ubuntu installation.

- Alternatively, Docker Desktop Community Edition for Mac can be used to downloand and install a Gadgetron Docker container.  After the Docker software installation is complete, and a network connection is available, the Gadgetron Docker container is installed by running the command:

```
   docker pull gadgetron/ubuntu_2004
```

Once the container has been downloaded and installed into the user's Docker environment, the user should then be able to launch it as they would any other container, for example:

```
   docker run gtLocal gadgetron/ubuntu_2004
```

and interact with the running container using any of Docker's standard suite of command line tools.

## Installing from Source Code

Compiling Gadgetron from sources is the traditional method of installation. It should be reasonably easy on most Linux distributions. You can find detailed instructions [here](https://github.com/gadgetron/gadgetron/wiki/Linux-Installation-(Gadgetron-4)). 

## Potential issue

### add-apt-repository: command not found

Fresh Ubutnu installations might not come with the `add-apt-repository` command installed. It's part of the `software-properties-common` package, and can simply be installed: 
```bash
sudo apt-get install software-properties-common
```

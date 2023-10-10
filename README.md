# Compiling Delft3D Flexible Mesh on Docker Containers

This repository contains the Dockerfiles to build containers to compile Delft3D Flexible Mesh on different OS <em>[source](https://oss.deltares.nl/web/delft3dfm/get-started#Download%20source%20code)</em>

Using >> ubuntu-20.04_intel_basic

Because of issues with SVN inside docker, the code is downloaded first (outside of docker), then copy inside container when building the image.

# General process for tag 68819
```
cd ./ubuntu-20.04_intel_basic
# install subversion
sudo apt-get -y install subversion
# pull the delft3dfm svn repositry - replace the username and password placeholders with your oss.deltares.nl credentials 
# For now we should use the tag 68819 as explained here : https://svn.oss.deltares.nl/repos/delft3d/tags/delft3dfm/142632/
# newer tags use a different install process (ongoing work to make it work, see below)
svn checkout https://svn.oss.deltares.nl/repos/delft3d/tags/delft3dfm/68819/ delft3dfm_68819 --username <username> --password <password>

# zip the repositry in folder for faster and reliable copying into docker-container filesystem and place it in the delft3dfm_source dir available for the docker instance during build
tar -czvf delft3dfm_68819.tar.gz delft3dfm_68819
mkdir delft3dfm_source && mv delft3dfm_68819.tar.gz delft3dfm_source/
# remove the downloaded repositry from host if desired
rm -rf delft3dfm_68819
```

Once complete we can build the docker image 

`"docker build --network=host -t delft3dfm:68819 ."`


To run interactively 

`"docker run --network=host -it delft3dfm:68819 ."`

Run the examples:

`cd /home/delft3dfm_compile/delft3dfm_68819/examples/01_standard`

The compiled binaries and shell scripts are saved in `/opt/delft3dfm/bin/***`
so paths in the bash scripts to run the examples (`run.sh` , `run_parallel.sh`) need to be updated

Change from 
```
#!/bin/bash
../../src/bin/run_dflow2d3d.sh
```

to 

```
#!/bin/bash
/opt/delft3dfm/bin/run_dflow2d3d.sh
```


# Push to Docker Hub 

```
docker tag delft3dfm:68819 simonwp/delft3dfm:68819 # tag local image with correct dockerhub name
docker push simonwp/delft3dfm:68819 # push image to docker hub
```

# General process for newer tags and/or trunk version (ongoing)

As outlined here https://svn.oss.deltares.nl/repos/delft3d/tags/delft3dfm/142632/, the build process is different for newer tags
using CMAKE

see src/README
cmake/README

`"docker build --network=host -t delft3dfm:140221 -f Dockerfile_CMAKE ."`


use build.sh that uses some of the build scripts in /src
trying ./build_gnu.sh
need to install gfortran `apt-get install gfortran` (see Dockerfile_CMAKE)

The bash scripts to run the different modules are now saved here : 
`/home/delft3dfm_compile/delft3dfm_140221/src/bin`

This path should be changed in bash scripts provided to run examples.
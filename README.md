# [WIP] Dockerize scripts for commercial EDA tools

I don't think dockerizing EDA tools is somewhat useful or not, but here is the `Dockerfile`s for dockerizing popular EDA tools!

## CAVEAT
  - I failed to achieve X11 forwarding. In other words, with this image you cannot open GUI windows.
  - Did not test uploading to "private" docker registry.
    - (you should not upload image containing commeiclal tool to public docker registry, of course ;-) )
  - Intermediate image does not removed automatically. You have to remove by yourself for now.
  - How can we run docker container with unpriviledged permission?
    - [Singularity](https://www.sylabs.io) could be a solution, but currently I failed to use that tool.

## Prerequisites
  - Docker 17.05+ (this script uses multi-stage build feature)
  - Installer and installation package of the tool you want to dockerize
     - This repository consists of just `Dockerfile`s, not of actual images or installer packages

## Generating an image
  
  - First, clone this repository to your workstation.
  - Copy (or bind mount) installer and installation package to the subdirectory
     - Refer to shell scripts for bind-mounting installation files from other path
  - modify dockerfile to match with your tool version, your requirements, etc.
  - Execute below command to create an image.  
`$ sudo docker build -t <image_name>:<version> -f <Dockerfile> .`  
`e.g. $ sudo docker build -t synopsys_dc:X-2020.4 -f Dockerfile_Synopsys_DC .`
  - Remove intermediate image created during building an image.
```bash
$ sudo docker images    # find a tag of the intermediate image
$ sudo docker rmi <image_tag_you_want_to_remove>
```

  - Example container launching command
    - You should pass environment variable regarding your license server
```bash
$ sudo docker run --rm -it -e LM_LICENSE_FILE="<port>@<license_server>" \
                   -env="DISPLAY" \
                   <image_name> [<command>]
```

## Vendor specific requirements

### Synopsys

 - Requires following jobs to be run within Ubuntu (14.04) environment
```bash
$ sudo apt install csh libxss1 libsm6 libice6 libxft2 libjpeg62 libtiff5 libmng2 libpng12-0
# WORKAROUND link old library filenames with newer version
$ sudo ln -s /usr/lib/x86_64-linux-gnu/libtiff.so.5 /usr/lib/x86_64-linux-gnu/libtiff.so.3
$ sudo ln -s /usr/lib/x86_64-linux-gnu/libmng.so.2 /usr/lib/x86_64-linux-gnu/libmng.so.1
```
 - Ubuntu 18.04 requires additional treatments, because `libpng12-0` package was removed from that version
   - manually download and install `libpng12-0` package, or add older source and install from that source like below:
```bash
$ sudo 'echo "deb http://security.ubuntu.com/ubuntu xenial-security main" >> /etc/apt/sources.list'
$ sudo apt update
$ sudo apt install -y -t xenial libpng12-0
```

### Cadence

 - Requires following packages
```bash
$ sudo apt install openjdk-6-jre  # for installer
$ sudo dpkg --add-architecture i386
$ sudo apt libxtst6:i386 libxext6:i386 libxi6:i386 ksh csh \
```

 - define below envvar to execute 64-bit binary  
`$ export CDS_AUTO_64BIT=ALL`

 - (Virtuoso) define below envvar regarding inside `<path/to/virtuoso>/share/oa/lib/`  
`$ export OA_UNSUPPORTED_PLAT "linux_rhel50_gcc48x"`


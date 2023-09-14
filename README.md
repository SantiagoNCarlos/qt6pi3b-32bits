# Cross-compiling Qt 6 for the Raspberry Pi 3B+ (32-bit)

## Preface

This is a guide for cross-compiling Qt 6 for Raspberry Pi 3B+ (32-bit OS). For a 64-bit cross compiling, check out [[1]](https://github.com/kevin-strobel/qt6pi3b).
This is a direct modification of [[1]](https://github.com/kevin-strobel/qt6pi3b), done by @kevin-strobel.
At the bottom I include a few steps on how to cross compile your project easily in case you're using qmake.

## Build

In the following, your "computer" refers to as where you execute Docker (most Linux distributions will do), "host" refers to as the Docker environment (Ubuntu 20.04 LTS), and "target" refers to as the Raspberry Pi 3B (Raspbian 64-bit).

### Raspberry Pi

- Setup the Raspberry Pi using a 32-bit image of Raspberry Pi OS.
- Install the required software

```
sudo apt update
sudo apt full-upgrade
sudo reboot

sudo apt-get install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev gstreamer1.0-omx libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev

sudo apt-get install libavcodec-dev libavformat-dev libswscale-dev libx11-dev freetds-dev libsqlite3-dev libpq-dev libiodbc2-dev firebird-dev libgst-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev libdirectfb-dev libaudio-dev libxkbcommon-x11-dev

sudo mkdir /usr/local/qt6
```

- Enable the SSH service and make sure that you can connect from your computer to your RPi.

### Computer

First of all, install *rsync*, *ssh*, *git* and *docker* on your computer. I assume you have a basic understanding of Docker.

Then,

- Checkout this repository
- Execute `./prepareSysroot.sh <RPI username> <RPI IP address>`
  This copies the Raspberry Pi's sysroot to your computer. Depending on your configuration, you may enter your RPi user's password three times.
- Carefully look at the Dockerfile's "*PLEASE CUSTOMIZE THIS SECTION*" and edit it if necessary. Edit the modules and include only what you need, this will reduce the build time. Have in consideration that, in case you need specific modules, maybe you'll have to install aditional software to your raspberry (For example, I used MariaDB and had to install the mariadb packages)
- Execute `docker build --tag qtpi/qtpi:1.0 .` (you may need to use `sudo`)
  This will generate a Docker image while compiling and cross-compiling Qt. Since most of the process is done here, it will take some time.
- When the last step succeeded, you now have the complete environment ready for compiling Qt applications for your host / your computer as well as your Raspberry Pi.
- At last, you should run a Docker container from the newly generated Docker image: `docker run -it --rm qtpi/qtpi:1.0`
  From there, simply execute `~/copyQtToRPi.sh <RPI username> <RPI IP address>`
  to copy the Qt files to your Raspberry Pi.

Inside the Docker container, the Qt host installation is located at **~/qt-host**, the Qt target installation at **~/qt-raspi** (see [[1]](https://github.com/kevin-strobel/qt6pi3b)).

## Cross-compiling your Qt project for the Raspberry Pi 3B+ with qmake (32-bit)

I used qmake for the cross compilation, so I do not know exactly how to cross compile using cmake, but you should find information about it in [[2]](https://github.com/PhysicsX/QTonRaspberryPi/blob/main/README.md) and [[3]](https://wiki.qt.io/Cross-Compile_Qt_6_for_Raspberry_Pi). 

Once you generated the docker image and copied the Qt files to your raspberry pi, you can cross compile your Qt project. 
 - First, you need to run a docker container from the Docker image you created before (in case you exit after copying Qt) with `docker run -it --rm qtpi/qtpi:1.0`
 - Open other terminal tab, and find the container ID with `docker ps`
 - Execute `docker cp /path/to/project container_id:/path/within/container` to copy your project to the docker container.
 - Navigate to your project's directory inside the Docker container with `cd /path/in/container`
 - Generate a Makefile for your Qt application. Use the `qmake` in `/qt-raspi/bin/`, this is the cross compiled version of `qmake`. Execute `/qt-raspi/bin/qmake your_project.pro`
 - Compile your Qt application using `make`. If all goes well, this should create an executable file in the current directory that is compatible with the Raspberry Pi.
 - Copy the compiled application back to your host machine executing `docker cp container_id:/path/in/container/build/executable /path/to/destination/on/host` (Execute in another terminal tab, outside docker container).
 - Exit the docker container.
 - Transfer the application to your Raspberry Pi with `scp /path/to/compiled/app pi@raspberry-pi-ip:/destination/path`

If all goes well, you should have your app running in your Raspberry Pi 3B+.

## References

[1] https://github.com/kevin-strobel/qt6pi3b

[2] https://github.com/PhysicsX/QTonRaspberryPi/blob/main/README.md

[3] https://wiki.qt.io/Cross-Compile_Qt_6_for_Raspberry_Pi

# Unofficial MongoDB Community Edition Binaries for Raspberry Pi

## Overview

These are a best-effort attempt to create binaries of the MongoDB Community Edition Server for the Raspberry Pi ecosystem. MongoDB Inc does not officially support these binaries.

## Motivation

[Time-Series Collections](https://www.mongodb.com/docs/v6.0/core/timeseries-collections/) are a relatively new MongoDB feature that provide a meaningful improvement for common embedded workloads, like sensor aggregation on a Raspberry Pi. Prior to this repo, users were required to build from source.

## Docker Support?

The binaries from this repo are packaged in a Docker container [here](https://github.com/themattman/mongodb-raspberrypi-docker).

## Pi Support

* Raspberry Pi 5: **New!!** Smoke-tested on real hardware. Beginning with [r7.0.3](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.3-rpi-unofficial).
* Raspberry Pi 4: Tested. I have this running on real hardware with an uptime greater than one year.
* Raspberry Pi 3: Untested, unlikely to work. May release a build in the future for this platform.

## Notes

MongoDB officially requires ARMv8.2-A+ [microarchitecture support](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-platform-considerations) as of MongoDB 5.0+. The Raspberry Pi 4 runs on ARMv8.0-A. These binaries are a best-effort at preserving functionality below minimum hardware specs.

However, the Raspberry Pi 5 does meet the minimum hardware requirements with its newer CPU.

These binaries are subject to the [MongoDB Server-Side Public License](https://github.com/mongodb/mongo/blob/r7.0.4/LICENSE-Community.txt).

## Releases

- [_r7.0.4_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.4-rpi-unofficial) [December 10, 2023]

- [_r7.0.3_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.3-rpi-unofficial) [November 11, 2023]

- [_r7.0.2_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.2-rpi-unofficial) [October 27, 2023]

- [_r6.0.11_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.11-rpi-unofficial) [October 25, 2023]

- [_r6.0.10_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.10-rpi-unofficial) [September 21, 2023]

- [_r7.0.1_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.1-rpi-unofficial) [September 18, 2023]

- [_r7.0.0_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.0-rpi-unofficial) [September 18, 2023]

- [_r6.0.8_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.8-rpi-unofficial) [August 24, 2023]

- [_r6.0.7_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.7-rpi-unofficial) [July 10, 2023]

- [_r6.0.5_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.5-rpi-unofficial) [April 10, 2023]

- [_r6.2.0_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.2.0-rpi-unofficial) [January 12, 2023]

- [_r6.1.0-rc4_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.1.0-rc4-rpi-unofficial) [October 3, 2022]


## Reproduction Build Instructions

- Instructions are for an Ubuntu 20.04 (focal) build environment. Ubuntu 18.04 was notably more difficult on my first attempt.

- Beginning August 2023 and after: Built using a cluster of 3 Raspberry Pi 4B's using [ice cream](https://github.com/icecc/icecream).

- Prior to August 2023: Built using [WSL2](https://learn.microsoft.com/en-us/windows/wsl/about) on Windows 10 with Intel Kaby Lake i7 CPU & 16GB RAM.

- Due to the hassle of installing special packages, it is recommended to set up a custom build environment using a VM, LXC container, Docker container, WSL, etc.

```
# Arm-Specific Cross-Compilation Instructions

$ sudo cat  >> /etc/apt/sources.list <<DEV
deb [arch=arm64] http://ports.ubuntu.com/ focal main multiverse universe
deb [arch=arm64] http://ports.ubuntu.com/ focal-security main multiverse universe
deb [arch=arm64] http://ports.ubuntu.com/ focal-backports main multiverse universe
deb [arch=arm64] http://ports.ubuntu.com/ focal-updates main multiverse universe

DEV
$ sudo dpkg --add-architecture arm64
$ sudo apt-get update || echo "continuing after 'apt-get update'"
$ compiler_version=10
$ sudo apt-get install -y gcc-${compiler_version}-aarch64-linux-gnu g++-${compiler_version}-aarch64-linux-gnu python3-venv
$ sudo apt-get install -y libssl-dev:arm64 libcurl4-openssl-dev:arm64 liblzma-dev:arm64

# MongoDB Instructions

$ git clone -b r7.0.4 git@github.com:mongodb/mongo.git r7.0.4
$ cd r7.0.4
$ python3 -m venv python3-venv
$ source python3-venv/bin/activate
$ python -m pip install "pip==21.0.1"
$ python -m pip install -r etc/pip/compile-requirements.txt
$ python -m pip install keyring jsonschema memory_profiler puremagic networkx cxxfilt

# Lowered -j to 1 below the number of cores available in order to not over-saturate build machine.
# This helps keep the system responsive to user input during the compile.

# The important part for cross-compilation is to specify the arm toolchain for the following three tools:
#  AR: Archive tool
#  CC: C compiler
# CXX: C++ compiler

# Should take less than a minute.
$ \time --verbose python3 buildscripts/scons.py -j$(($(grep -v processor /proc/cpuinfo)-1)) AR=/usr/bin/aarch64-linux-gnu-ar CC=/usr/bin/aarch64-linux-gnu-gcc-${compiler_version} CXX=/usr/bin/aarch64-linux-gnu-g++-${compiler_version} CCFLAGS="-march=armv8-a+crc -moutline-atomics -mtune=cortex-a72" --dbg=off --opt=on --link-model=static --disable-warnings-as-errors --ninja generate-ninja NINJA_PREFIX=aarch64_gcc_s VARIANT_DIR=aarch64_gcc_s DESTDIR=aarch64_gcc_s

# Will take several hours and depends heavily on your machine's capabilities. Almost 4 hours on my machine.
$ \time --verbose ninja -f aarch64_gcc_s.ninja -j$(($(grep -c processor /proc/cpuinfo)-1)) install-devcore # For MongoDB 6.x+

# Minimize size of executables for embedded use by removing symbols
pushd aarch64_gcc_s/bin
$ mv mongo mongo.debug
$ mv mongod mongod.debug
$ mv mongos mongos.debug
$ aarch64-linux-gnu-strip mongo.debug -o mongo
$ aarch64-linux-gnu-strip mongod.debug -o mongod
$ aarch64-linux-gnu-strip mongos.debug -o mongos

# Generate release (on Mac OS)
$ tar --gname root --uname root -czvf mongodb.ce.pi4.r7.0.4.tar.gz LICENSE-Community.txt README.md mongo{d,,s}
# Generate release (on Linux)
$ tar --group root --owner root -czvf mongodb.ce.pi4.r7.0.4.tar.gz LICENSE-Community.txt README.md mongo{d,,s}
```

## Installing on Raspberry Pi

- Ensure Raspberry Pi meets minimum HW requirements. I have only installed on a 4GB/8GB Raspberry Pi 4 & 8GB Raspberry Pi 5. Unknown how Pi's with lower specs will fare.

- Ensure a [64-bit Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) has been installed on the Pi.

  - Raspbian is 32-bit by default for maximum compatibility.

```
# Using wget assumes network connection. Can also copy with USB.
$ mkdir ~/mdb-binaries && cd ~/mdb-binaries
$ wget https://github.com/themattman/mongodb-raspberrypi-binaries/releases/download/r7.0.4-rpi-unofficial/mongodb.ce.pi4.r7.0.4.tar.gz
$ tar xzvf mongodb.ce.pi4.r7.0.4.tar.gz # Decompress tarball

# Prepare MongoDB data & log directories
$ mkdir -p /data/db/test_db
$ touch /data/db/test_db/mongod.log
$ sudo chown -R ${USER}:${USER} /data

# Run & Configure MongoDB Standalone Local Server
$ ./mongod --dbpath /data/db/test_db --fork --logpath /data/db/test_db/mongod.log --port 28080
$ ./mongo --port 28080 # run queries!
```

## Bugs / Requests

File an [issue](https://github.com/themattman/mongodb-raspberrypi-binaries/issues) on Github. Please include:

- hardware details

- steps you've tried

- error output

- general feedback

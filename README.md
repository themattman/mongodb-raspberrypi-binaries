# Unofficial MongoDB Community Edition Binaries for Raspberry Pi

## Overview

These are a best-effort attempt to create binaries of the MongoDB Community

Edition Server for the Raspberry Pi ecosystem. MongoDB Inc does not officially

support these binaries.

## Motivation

[Time-Series Collections](https://www.mongodb.com/docs/v6.0/core/timeseries-collections/) are a relatively new MongoDB feature that provide a

meaningful improvement for common embedded workloads, like sensor aggregation on

a Raspberry Pi. Prior to this repo, users were required to build from source.

## Notes

MongoDB requires ARMv8.2-A+ [microarchitecture support](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-platform-considerations) as of MongoDB 5.0+.

These binaries are subject to the [MongoDB Server-Side Public License](https://github.com/mongodb/mongo/blob/r6.2.0/LICENSE-Community.txt).

## Releases

- _r6.2.0_ [January 12, 2023]

  - mongo  - legacy MongoDB shell

  - mongod - MongoDB Community Edition Server

  - mongos - MongoDB Query Router - for [Sharded Clusters](https://www.mongodb.com/docs/manual/sharding/)

- _r6.1.0-rc4_ [October 3, 2022]

  - mongo  - legacy MongoDB shell

  - mongod - MongoDB Community Edition Server

  - mongos - MongoDB Query Router - for [Sharded Clusters](https://www.mongodb.com/docs/manual/sharding/)

## Reproduction Build Instructions

Instructions are for an Ubuntu 20.04 (focal) build environment. Ubuntu 18.04 was

notably more difficult on my first attempt.

Built using [WSL2](https://learn.microsoft.com/en-us/windows/wsl/about) on Windows 10 with Intel Kaby Lake i7 CPU & 16GB RAM.

Due to the hassle of installing special packages, it is recommended to set up

a custom build environment using a VM, LXC container, Docker container, WSL,

etc.

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
$ sudo apt-get install -y gcc-8-aarch64-linux-gnu g++-8-aarch64-linux-gnu ninja-build python3-venv
$ sudo apt-get install -y libssl-dev:arm64 libcurl4-openssl-dev:arm64 liblzma-dev:arm64

# MongoDB Instructions

$ git clone -b r6.2.0 git@github.com:mongodb/mongo.git r6.2.0
$ cd r6.2.0
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
$ \time --verbose python3 buildscripts/scons.py -j$(($(grep -c processor /proc/cpuinfo)-1)) AR=/usr/bin/aarch64-linux-gnu-ar CC=/usr/bin/aarch64-linux-gnu-gcc-8 CXX=/usr/bin/aarch64-linux-gnu-g++-8 CCFLAGS="-march=armv8-a+crc -mtune=cortex-a72" --dbg=off --opt=on --link-model=static --disable-warnings-as-errors --ninja generate-ninja NINJA_PREFIX=aarch64_gcc_s

# Will take several hours and depends heavily on your machine's capabilities. Almost 4 hours on my machine.
$ \time --verbose ninja -f aarch64_gcc_s.ninja -j$(($(grep -c processor /proc/cpuinfo)-1)) install-devcore # For MongoDB 6.x+
$ \time --verbose ninja -f aarch64_gcc_s.ninja -j$(($(grep -c processor /proc/cpuinfo)-1)) install-core    # For MongoDB 5.x

# Minimize size of executables for embedded use by removing symbols
$ mv build/install/bin/mongo build/install/bin/mongo.debug
$ mv build/install/bin/mongod build/install/bin/mongod.debug
$ mv build/install/bin/mongos build/install/bin/mongos.debug
$ aarch64-linux-gnu-strip build/install/bin/mongo.debug -o build/install/bin/mongo
$ aarch64-linux-gnu-strip build/install/bin/mongod.debug -o build/install/bin/mongod
$ aarch64-linux-gnu-strip build/install/bin/mongos.debug -o build/install/bin/mongos

# Generate release (on Mac OS)
$ tar --gname root --uname root -czvf mongodb.ce.pi.r6.2.0.tar.gz LICENSE-Community.txt README.md mongo{d,,s}
```

## Installing on Raspberry Pi

- Ensure Raspberry Pi meets minimum HW requirements. Must be 4+, 4GB RAM+.

- Ensure a [64-bit Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) has been installed on the Pi.

  - Raspbian is 32-bit by default for maximum compatibility.

```
# Using wget assumes network connection. Can also copy with USB.
$ mkdir ~/mdb-binaries && cd ~/mdb-binaries
$ wget https://github.com/themattman/mongodb-raspberrypi-binaries/releases/download/r6.2.0-rpi-unofficial/mongodb.ce.pi.r6.2.0.tar.gz
$ tar xzvf mongodb.ce.pi.r6.2.0.tar.gz # Decompress tarball

# Prepare MongoDB data & log directories
$ mkdir -p /data/db/ts_test_db
$ mkdir -p /data/db/ts_test_db_log
$ sudo chown -R ${USER}:${USER} /data

# Run & Configure MongoDB Standalone Local Server
$ ./mongod --dbpath /data/db/ts_test_db --fork --logpath /data/db/ts_test_db_log --port 28080
$ ./mongo --port 28080 # run queries!
```

## Bugs / Requests

File an [issue](https://github.com/themattman/mongodb-raspberrypi-binaries/issues) on Github. Please include:

- hardware details

- steps you've tried

- error output

- general feedback

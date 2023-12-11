# Reproduction Build Instructions

- Instructions are for an Ubuntu 20.04 (focal) build environment. Ubuntu 18.04 was notably more difficult on my first attempt.

- In the Fall of 2023, these releases are now being built on an individual Raspbery Pi 4 or Raspberr y Pi 5 with 8GB of RAM. Separate Pi's are dedicated to different major release branches.

- Beginning August 2023 and after: Built using a cluster of 3 Raspberry Pi 4B's using [ice cream](https://github.com/icecc/icecream).

- Prior to August 2023: Built using [WSL2](https://learn.microsoft.com/en-us/windows/wsl/about) on Windows 10 with Intel Kaby Lake i7 CPU & 16GB RAM.

- Due to the hassle of installing special packages, it is recommended to set up a custom build environment using a VM, LXC container, Docker container, WSL, etc.

The instructions below were developed for MongoDB 6.0 on the Pi 4. Slight adjustments are needed for other major release versions and other hardware platforms.

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

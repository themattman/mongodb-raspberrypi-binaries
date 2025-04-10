# Unofficial MongoDB Community Edition Binaries for Raspberry Pi

> [!NOTE]
> The Raspberry Pi 5 is officially supported by Mongodb Inc. New releases after 8.0.4 will ***only*** be compiled for Raspberry Pi 4 on a best effort basis.

## Overview

This repository provides best-effort binaries of the MongoDB Community Edition Server for Raspberry Pi. These are not officially supported by MongoDB Inc.

Earlier versions (MongoDB 6.x and 7.x) were maintained by [@themattman](https://github.com/themattman). My contributions start with MongoDB 8.0.4 for Raspberry Pi 5.

## Motivation

[Time-Series Collections](https://www.mongodb.com/docs/v6.0/core/timeseries-collections/) are especially useful for sensor workloads—common in Raspberry Pi projects. Building MongoDB from source used to be the only option. This project provides ready-to-run binaries instead.

## Pi Support

- **Raspberry Pi 5:** Compiled and tested [8.0.4](https://github.com/tototomate123/mongodb-raspberrypi-binaries/releases/tag/r8.0.4-rpi5) directly on a Pi 5. Since official MongoDB support now exists for this board, no future Pi 5 builds are planned unless requested. This release also includes optional install and uninstall scripts.

- **Raspberry Pi 4:** Builds are maintained by themattman and have been tested extensively. I plan to provide an 8.x build for Pi 4 soon.

- **Raspberry Pi 3:** Not supported and untested.

## Notes

MongoDB requires ARMv8.2-A+ as of version 5.0. Raspberry Pi 4 (ARMv8.0-A) is below spec, but unofficial builds work in practice. Raspberry Pi 5 meets all official requirements.

These binaries are released under the [MongoDB Server-Side Public License](https://github.com/mongodb/mongo/blob/r7.0.14/LICENSE-Community.txt).

## My Releases

### Raspberry Pi 5

- [r8.0.4](https://github.com/tototomate123/mongodb-raspberrypi-binaries/releases/tag/r8.0.4-rpi5) – April 6, 2025
  - Last unofficial Pi 5 build unless requests arise. Includes optional install and uninstall scripts. Use official binaries going forward or open an issue for feedback.

### Raspberry Pi 4

- r8.0.4 - coming soon

### Older binaries for Pi 4 & 5 (compiled by @themattman)

- [r7.0.14](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.14-rpi-unofficial) – Sept 17, 2024
- [r7.0.11](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.11-rpi-unofficial) – June 3, 2024
- [r7.0.9](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.9-rpi-unofficial) – April 30, 2024

## Installing on Raspberry Pi
> [!NOTE]
> This is only for @themattman binaries.

Use a 64-bit Raspberry Pi OS. Default Raspbian is 32-bit and not compatible.

```bash
# Download & extract binary
mkdir ~/mdb-binaries && cd ~/mdb-binaries

# Example for Raspberry Pi 4 (older release by themattman):
wget https://github.com/themattman/mongodb-raspberrypi-binaries/releases/download/r7.0.14-rpi-unofficial/mongodb.ce.pi4.r7.0.14.tar.gz

# Or for Raspberry Pi 5:
# wget https://github.com/tototomate123/mongodb-raspberrypi-binaries/releases/download/r8.0.4-rpi5/mongodb.ce.pi5.r8.0.4.tar.gz

# Extract
tar xzvf mongodb.ce.pi4.r7.0.14.tar.gz

# Set up DB path and logs
sudo mkdir -p /data/db/test_db
sudo touch /data/db/test_db/mongod.log
sudo chown -R $(whoami):$(whoami) /data

# Run MongoDB
./mongod --dbpath /data/db/test_db --fork --logpath /data/db/test_db/mongod.log --port 28080

# Connect
./mongo --port 28080
```

## Questions or Issues?

Please [open an issue](https://github.com/themattman/mongodb-raspberrypi-binaries/issues) and include:

- Raspberry Pi model
- OS details
- Any error output
- What you’ve tried so far

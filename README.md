# Unofficial MongoDB Community Edition Binaries for Raspberry Pi

## Overview

These are a best-effort attempt to create binaries of the MongoDB Community Edition Server for the Raspberry Pi ecosystem. MongoDB Inc does not officially support these binaries.

## Motivation

[Time-Series Collections](https://www.mongodb.com/docs/v6.0/core/timeseries-collections/) are a relatively new MongoDB feature that provide a meaningful improvement for common embedded workloads, like sensor aggregation on a Raspberry Pi. Prior to this repo, users were required to build from source.

## Docker Support?

The binaries from this repo are packaged in a Docker container [here](https://github.com/themattman/mongodb-raspberrypi-docker).

## Pi Support

* Raspberry Pi 5: **New!!** Smoke-tested on real hardware. Beginning with [r7.0.3](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.3-rpi-unofficial) and [r6.0.12](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.12-rpi-unofficial).
* Raspberry Pi 4: Tested. I have this running on real hardware with an uptime greater than one year.
* Raspberry Pi 3: Untested, unlikely to work. May release a build in the future for this platform.

## Notes

MongoDB officially requires ARMv8.2-A+ [microarchitecture support](https://www.mongodb.com/docs/manual/administration/production-notes/#std-label-prod-notes-platform-considerations) as of MongoDB 5.0+. The Raspberry Pi 4 runs on ARMv8.0-A. These binaries are a best-effort at preserving functionality below minimum hardware specs. For further digging, see [WT-9256](https://jira.mongodb.org/browse/WT-9256) for understanding the risks of running MongoDB 5.0+ on a Pi 4 (i.e. an ARM CPU without Large System Extensions (LSE) atomics).

However, the Raspberry Pi 5 does meet the minimum hardware requirements with its newer CPU and I would encourage users to seek the MongoDB official binaries for ARM that are compatible with the Pi 5.

These binaries are subject to the [MongoDB Server-Side Public License](https://github.com/mongodb/mongo/blob/r7.0.14/LICENSE-Community.txt).

## Releases

- [_r7.0.14_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.14-rpi-unofficial) [September 17, 2024]

- [_r7.0.11_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.11-rpi-unofficial) [June 03, 2024]

- [_r7.0.9_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.9-rpi-unofficial) [April 30, 2024]

- [_r7.0.8_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.8-rpi-unofficial) [April 22, 2024]

- [_r7.0.8_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.8-rpi-unofficial) [April 11, 2024]

- [_r7.0.7_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.7-rpi-unofficial) [April 03, 2024]

- [_r7.0.7_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.7-rpi-unofficial) [April 01, 2024]

- [_r7.0.6_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.6-rpi-unofficial) [March 21, 2024]

- [_r6.0.13_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.13-rpi-unofficial) [January 29, 2024]

- [_r7.0.5_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r7.0.5-rpi-unofficial) [January 06, 2024]

- [_r6.0.12_](https://github.com/themattman/mongodb-raspberrypi-binaries/releases/tag/r6.0.12-rpi-unofficial) [January 03, 2024]

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

## Installing on Raspberry Pi

- Ensure Raspberry Pi meets minimum HW requirements. I have only installed on a 4GB/8GB Raspberry Pi 4 & 8GB Raspberry Pi 5. Unknown how Pi's with lower specs will fare.

- Ensure a [64-bit Raspberry Pi OS](https://www.raspberrypi.com/software/operating-systems/) has been installed on the Pi.

  - Raspbian is 32-bit by default for maximum compatibility.

```
# Using wget assumes network connection. Can also copy with USB.
$ mkdir ~/mdb-binaries && cd ~/mdb-binaries
$ wget https://github.com/themattman/mongodb-raspberrypi-binaries/releases/download/r7.0.14-rpi-unofficial/mongodb.ce.pi4.r7.0.14.tar.gz
$ tar xzvf mongodb.ce.pi4.r7.0.14.tar.gz # Decompress tarball

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

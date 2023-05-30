# cephfs-bench
A script suite for creating continuous repetitive load on a file system with timings.

This is a heavily tailored collection of scripts for long-term testing a ceph fs file system. It aims at simulating a mix of realistic workloads with focus to specific bad corner cases that are known to stress ceph fs. Be aware that this suite contains hard-coded paths and docker-exec commands specific to our own deployment. Some editing will be required to run this suite on a different installation.

## Getting started

There is no documentation other than the code. Here some points that hopefully help:

- The suite implements a daemon-like constantly running process that can be configured while running.
- A simple FIFO communication protocol is implemented. This even allows to update the main script itself without interrupting the load producing process.
- There are a number of parameters that influence how much data and how many snapshots are kept on the cluster at all times: `max-blobs` and the command sequences for each loop `blob-add-cmds` and `blob-del-cmds` initialised in the main script `bench`.
- The basic idea is to run a loop over `blob-create` and `blob-delete` commands that create and delete data. In between snapshots are created and deleted as configured. Hence it is possible to control how much data will accumulate in total and how big each snapshot is. A slow ramping up `max-blobs` is recommended due to excessive over-allocation with this particularly bad workload.

## Entry point

- Everything is configured to be executed by a user `ansible` that has password-less sudo permission.
- Entry point of the suite is the script `bench-start`; this will start a daemonised instance of `bench`.
- The main loop is in `bench`.
- I'm afraid you will need to read through the files to figure out how everything is glued together.
- Important status collection is in commands starting with "docker exec". These need to be adjusted to your deployment.
- Scripts that don't show up in other scripts are usually for communicating with the daemon or querying status info.

## What else do I need?

The suite expects two files in the home dir of user `ansible`: `conda.tgz` and `image.iso`. To have reasonably clean timings, both files are copied to a ram disk in `bench-prepare`. For an ISO image, just download any image that is at least 2GB. It should have a copy time of at least 10s on a non-busy cluster.

For creating a `conda.tgz` archive, follow the [instructions for installing conda](https://docs.anaconda.com/free/anaconda/install/linux/), make sure you don't allow conda to initialise anything at the end of the install and archive the folder created. Currently the archive is expected to be compressed, but for higher IOP/s pressure you can change the scripts to use an un-compressed archive. The un-tar (and later rm) of the conda archive is particularly challenging for ceph because it is very IOP/s intensive and it creates about 90% of all files as hard links.

It would be good to add a typical HPC parallel build task. This is where users feel the latency of ceph compared with other network file systems.

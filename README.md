# MLEC Basic
Basic configuration for Multi-Level Erasure Coding (MLEC)

In this guide, we configure an MLEC cluster on a single machine to show the basic configuration and usage of MLEC.

We use HDFS as the top-level erasure-coded system, and ZFS as the bottom-level erasure-coded file system.

We configure HDFS using docker. Each HDFS datanode is running in a separate docker container.

We configure ZFS using loopback devices as *fake* disks. This is because a MLEC cluster requires many disks to perform the multi-level erasure coding, while a cloud machine usually doesn't have enough disks for our experiment. Therefore, we create 9 loopback devices on memory to mimic disks.

The following steps have been tested on a Ubuntu 20 machine on Chameleon Cloud.

### Set up ssh keys for github

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts
cat ~/.ssh/id_rsa.pub
```

Copy and paste the public key into: https://github.com/settings/keys

### Configure your username and email.
```
git config --global user.name "FIRST_NAME LAST_NAME"
git config --global user.email "MY_NAME@example.com"
```

for example:

```
git config --global user.name "Meng Wang"
git config --global user.email "mengwanguc@gmail.com"
```

### Clone this repo

```
git clone https://github.com/mengwanguc/mlec-basic
```

## Set up ZFS

Check [zfs-setup/README.md](zfs-setup/README.md) and view the readme there.

## Set up HDFS

Check [hdfs-setup/README.md](hdfs-setup/README.md) and view the readme there.
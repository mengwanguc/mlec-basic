## ZFS set up

### Install the required packages

```
sudo apt-get update
sudo apt install build-essential autoconf automake libtool gawk alien fakeroot \
    dkms libblkid-dev uuid-dev libudev-dev libssl-dev zlib1g-dev libaio-dev \
    libattr1-dev libelf-dev linux-headers-generic python3 python3-dev \
    python3-setuptools python3-cffi libffi-dev python3-packaging git \
    libcurl4-openssl-dev debhelper-compat dh-python po-debconf python3-all-dev \
    python3-sphinx

sudo apt-get install linux-image-generic
```

### Install OpenZFS

```
cd ~
git clone https://github.com/openzfs/zfs.git
cd zfs
```

Install
```
sh autogen.sh
 ./configure
make -s -j$(nproc)

sudo make install
sudo ldconfig
sudo depmod

sudo ./scripts/zfs.sh 
sudo systemctl enable zfs.target zfs-import.target \
    zfs-mount.service zfs-import-cache.service zfs-import-scan.service
```


You can see whether zfs is successfully installed by trying the command "zfs list". This the expected result:

```
cc@node2:~/zfs-MLEC$ zfs list
no datasets available
```

### Create a 2+1 local zfs pool

Come back to this repo:

```
cd ~/mlec-basic/zfs-setup/
```

A ZFS pool requires multiple disks. Because we usually don't have that many disks on most Cloud servers, here
we create *fake* disks using loopback devices.

Here we create 9 loopback devices. Each loopback device is 1GB and is on memory. Therefore, it requires 9GB memory.
Please update the script to use less device size if your server doesn't have enough memory.

Press enter for each options pop up

```
sudo bash create_loopback.sh
```

Set up a 2+1 zfs pool:

```
mkdir -p /home/cc/hdfs
sudo zpool create pool1 raidz /dev/loop21 /dev/loop22 /dev/loop23
sudo zfs set mountpoint=/home/cc/hdfs/zfs-folder-1 pool1
sudo chown -R cc:cc /home/cc/hdfs/zfs-folder-1
```

Note that the above commands assume your user is `cc`.

Check zpool status:

```
sudo zpool status
```

### Prepare 3 ZFS folders

We have already created one zpool. Let's now create another 2 zpools:

```
sudo zpool create pool2 raidz /dev/loop24 /dev/loop25 /dev/loop26
sudo zfs set mountpoint=/home/cc/hdfs/zfs-folder-2 pool2
sudo chown -R cc:cc /home/cc/hdfs/zfs-folder-2
```

```
sudo zpool create pool3 raidz /dev/loop27 /dev/loop28 /dev/loop29
sudo zfs set mountpoint=/home/cc/hdfs/zfs-folder-3 pool3
sudo chown -R cc:cc /home/cc/hdfs/zfs-folder-3
```

Check zpool status. You should see 3 pools:

```
sudo zpool status
```
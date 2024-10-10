# Set up HDFS

Set up HDFS using docker.

Here we show how to set up a HDFS cluster using 2+1 erasure coding.

## Install docker

```
sudo apt update
sudo apt upgrade -y
sudo apt install -y docker.io
```

Start the docker service

```
sudo systemctl start docker
sudo systemctl enable docker
```

Check docker version (so that we confirm docker is installed properly)
```
docker --version
```

## Install docker-compose

Download the latest version of Docker Compose:

```
sudo curl -L "https://github.com/docker/compose/releases/download/$(curl -s https://api.github.com/repos/docker/compose/releases/latest | grep -Po '"tag_name": "\K.*?(?=")')/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

Set executable permissions:

```
sudo chmod +x /usr/local/bin/docker-compose
```

Verify Docker Compose installation:
```
docker-compose --version
```

## Run the containers

*Make sure* you have 3 zfs pools mounted to the following folders:
```
/home/cc/hdfs/zfs-folder-1
/home/cc/hdfs/zfs-folder-2
/home/cc/hdfs/zfs-folder-3
```

Start the containers:
```
sudo docker-compose up -d
```

### Enable erasure coding in HDFS:

Enter the NameNode container:
```
sudo docker exec -it namenode bash
```

Create a directory for erasure-coded data:
```
hdfs dfs -mkdir /ec
```

Enable Erasure Coding Policy (e.g., XOR-2-1-1024k):
```
hdfs ec -enablePolicy -policy XOR-2-1-1024k
```

Set the EC policy to be 2+1:
```
hdfs ec -setPolicy -path /ec -policy XOR-2-1-1024k
```

### Verify the erasure coding
Check the EC policy:
```
hdfs ec -getPolicy -path /ec
```

Create a random 10MB file and write it into hdfs with block size 1MB:
```
cd /
dd if=/dev/urandom of=randomfile.txt bs=1M count=10

# write to hdfs with block size = 1MB
hdfs dfs -D dfs.blocksize=1048576 -put randomfile.txt /ec
```

Verify the file placement:
```
hdfs fsck /ec/randomfile.txt -files -blocks -locations
```

You will see the file is stored as 5 stripes, with each stripe of 2 MB (2*1MB data block + 1 parity block).

You can also go to the datanode and check the actual block data there.

First exit the namenode (Ctrl+D), and then run:
```
sudo docker exec -it datanode1 bash
```

In the datanode1, run
```
du -sh hadoop/dfs/data/current/
```

You can see the actual data stored in datanode1 is 5MB. Similarly, datanode2 has 5MB and datanode3 has 5MB. All adds up to 10MB data and 5MB parity.

You can also verify it on the host machine:
```
sudo du -sh * /home/cc/hdfs/zfs-folder-1/current/
```

You can also veriy it by checking zpool:

```
sudo zpool list
```

The expected output is:

```
cc@meng-mlec-sim:~/hdfs$ sudo zpool list
NAME    SIZE  ALLOC   FREE  CKPOINT  EXPANDSZ   FRAG    CAP  DEDUP    HEALTH  ALTROOT
pool1  2.75G  7.88M  2.74G        -         -     0%     0%  1.00x    ONLINE  -
pool2  2.75G  7.85M  2.74G        -         -     0%     0%  1.00x    ONLINE  -
pool3  2.75G  7.85M  2.74G        -         -     0%     0%  1.00x    ONLINE  -
```

The ALLOC is 7.88MB. It is 1.5*5MB, because each ZFS pools stores 5MB block from HDFS, and does further 2+1 in ZFS in the 2nd level, which creates another 2.5MB parity.

You might notice the size is slightly higher than expected (e.g. 7.88MB > 7.5MB). This happens for both HDFS and ZFS, probably because
it needs some extra space for storing metadata etc.
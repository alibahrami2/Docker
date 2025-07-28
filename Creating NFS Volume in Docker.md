# Creating NFS Volume in Docker 🐋💾



📝 Introduction

Docker volumes are the preferred mechanism for setting up persistent storage for your Docker containers. Volumes are existing directories on the host filesystem mounted inside a container. They can be accessed both from the container and the host system.

Docker also allows users to mount directories shared over the NFS remote file-sharing system. The volumes created for this purpose use Docker's own NFS driver, eliminating the need to mount the NFS directory on the host system.

This tutorial will show you how to create and use NFS Docker Volumes.

## Pre-requirements📑
⚠️ We need 2 VMs

🔸First VM is for Docker

🔸Second VM uses as a NFS Volume Driver


💻 IP VM-01 (Docker) : `192.168.1.103`🐋

💻 IP VM-02 (NFS) : `192.168.1.104`🐧

## 1. Configuring second VM for NFS Volume📑
🔺First we should install this package : `nfs-kernel-server`

```bash 
sudo apt install nfs-kernel-server
```

🔺Then we should configure `/etc/exports` for NFS Volume so we can create a view files and directories form our Docker-VM (VM-01)

```bash
vim /etc/exports
```

🔺Configuring /etc/exports
```bash
/data 192.168.1.103(rw,sync,no_subtree_check,no_root_squash)
```

Based on this config , we do not worry about any IP in our network to work !

🔺We have one directory `/data` in our second VM . So we should create this directory !
```bash
mkdir /data
```

After configuring the exports, apply the changes by running the exportfs command:

```bash
exportfs -a
```

**************

⚠️ If you use firewall in your VMs , you should configure it to allow these IP to communicate . ⚠️

First of all, check `ufw status`, if it was inactive, you most active ufw with `ufw enable` command.

When you have the UFW firewall enabled, you will need to allow traffic on the NFS port `2049`:

`ufw allow from client_ip to any port nfs`

```bash
ufw allow from 192.168.1.103 to any port 2049
```
**************

## 2. Configuring First VM (Docker)
🕹 In this section we can use two methods for NFS volume creation

🔸Method 1 : First, create volume and then mount it to the container based on a image

🔸Method 2 : Use `dokcer run` for volume creation and container simultaneously We check these two methods separately

First we need to install `nfs-common` on our dockerhost.
## 2.1. install nfs-common package on docker host


🗳 To mount the NFS volume into a container, install the nfs-common package on the host system.

🔺Start by updating the repositories.
```bash
sudo apt update
```
🔺Use APT to install the `nfs-common` package.
```bash
sudo apt install nfs-common
```
## 2.2. Method_1
First create a NFS Volume. The simplest way to create and manage Docker volumes is using the docker volume command and its subcommands.

The syntax for creating an NFS Docker volume includes two options.

1. The `--driver` option defines the local volume driver, which accepts options similar to the mount command in Linux.
2. The `--opt` option is called multiple times to provide further details about the volume.
The details include:

  🔹 The volume type.

  🔹 The write mode.

  🔹 The IP or web address of the remote NFS server.
 
  🔹 The path to the shared directory on the server.
```bash
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=[nfs-ip-address],rw \
  --opt device=:[path-to-directory] \
  [volume-name]
```
The example below illustrates creating an NFS Docker volume named nfs-volume. The volume contains the `/data` directory located on the server, with the `rw` (read/write) permission. The IP address of the server is `192.168.1.104`.

The successfully executed command outputs the name of the volume.
```bash
docker volume create --driver local \
  --opt type=nfs \
  --opt o=addr=192.168.1.104,rw \
  --opt device=:/data \
  nfs-volume
```
## Mount NFS in a Container
```bash
docker run -d -it \
  --name [container-name] \
  --mount source=[volume-name],target=[mount-point]\
  [image-name]
```
The example below mounts the NFS volume named `nfs-volume` to the `/data` directory in the container.

```bash
docker run -d -it \
  --name nfs-container \
  --mount source=nfs-volume,target=/data\
  centos:latest
```
Enter the container environment bash shell with docker exec:
```bash
docker exec -it nfs-container /bin/bash
```
If you create a file inside the Docker container, it will also be accessible in the original directory on the server. To test, use the touch command to create an empty file in the /mnt directory.

```bash
touch /data/file01.txt
```

On the server, navigate to the directory you shared and list its contents. The file created in the Docker container appears.
```usage
root@nfs-server:~# ls /data
#OUT PUT

file01.txt
```
## 2.3 Method 2
In this method we use `docker run` to create volume and container simultaneously
So :
```bash
docker run -itd \
  --name CONTAINER_NAME \
  --mount 'type=volume,source=NFS_VOLUME_NAME,dst=/data
  --volume-driver=local \
  --volume-opt=type=nfs \
  --volume-opt=device=:/app \
  --volume-opt=o=addr=NFS_IP' \
  centos:latest
```

For example based on our scenario :
```bash
docker run -itd \
  --name nfs-container \
  --mount 'type=volume,source=nfs-volume,dst=/data
  --volume-driver=local \
  --volume-opt=type=nfs \
  --volume-opt=device=:/data \
  --volume-opt=o=addr=192.168.1.104' \
  centos:latest
```
We can check our method based on the previous one.




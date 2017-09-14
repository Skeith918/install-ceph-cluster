# Install-ceph-cluster

This is an tutorial to install a ceph cluster

## My infrastructure

##### 3 servers for storage :
- 3 disks on each server : 1 disk for system and 1 monitor, 1 disk for OSD = 2 OSD per servers = 6 disks and OSDs
- A configuration to have a margin of loss of 2 disks out of 6.

##### 2 server for gateway
##### 1 server for admin

All servers working on Debian 8 and have a domain name.

- files-01.example.com files-02.example.com files-03.example.com
- files-rgw-01.example.com fles-rgw-02.example.com
- files-admin.example.com

On storage servers, 1 disk is already used for system, the 2 others are not partitionned.
 
## Installation

##### On Admin

- Generate ssh key pair without passphrase
```
ssh-keygen
```
- Add your public key in all server's authorized_keys
- Configure your servers to connect to him with ssh-key
- Create hostname redirection in your /etc/hosts for all your servers 
```
1.1.1.1   files-01
2.2.2.2   files-02
3.3.3.3   files-03
1.1.2.1   files-rgw-01
1.1.2.2   files-rgw-02
```
- Add release key
```
wget -q -O- 'https://download.ceph.com/keys/release.asc' | sudo apt-key add -
```
- Add the Ceph luminous packages to your repository (lasted at September 2017)
```
echo deb https://download.ceph.com/debian-luminous/ jessie main | sudo tee /etc/apt/sources.list.d/ceph.list
```
- Install package
```
apt update
apt install ceph-deploy
```
- Create a directory in your home path
```
mkdir ~/ceph-cluster
```
- Note that you must be in this directory to use ceph-deploy command
- In your ceph-cluster directory, create your cluster with your 3 storage servers
```
ceph-deploy new files-01 files-02 files-03
```
- Install ceph luminous release in your servers
```
ceph-deploy install --release luminous files-01 files-02 files-03
```
- Deploy the monitors in your servers and create keys
```
ceph-deploy mon create-initial
```
- Deploy the keys in your servers
```
ceph-deploy admin files-01 files-02 files-03
```
- Deploy an manager (use the server you want, i use the files-01 server)
```
ceph-deploy mgr create files-01
```
- Delete partition table of the not used disks in storage servers (in my case /dev/sdb and /dev/sdc on each server)
```
ceph-deploy disk zap files-01:sdb files-01:sdc files-02:sdb files-02:sdc files-03:sdb files-03:sdc
```
- Create the osd 
```
ceph-deploy osd create files-01:sdb files-01:sdc files-02:sdb files-02:sdc files-03:sdb files-03:sdc
```
- 

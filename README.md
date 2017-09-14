# Install-ceph-cluster

This is an tutorial to install a ceph cluster

## My infrastructure

- 3 servers for storage (3 disks : 1 disk for system and 1 monitor, 1 disk for OSD = 2 OSD per servers)
- 2 server for gateway
- 1 server for admin

All servers working on Debian 8.
All servers have a domain name.

- files-01.example.com files-02.example.com files-03.example.com
- files-rgw-01.example.com fles-rgw-02.example.com
- files-admin.example.com

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


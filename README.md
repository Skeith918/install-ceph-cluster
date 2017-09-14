# Install-ceph-cluster

This is an tutorial to install a ceph cluster

### My infrastructure

- 3 servers for storage (1 disk is already used for system, the 2 others are not used and partitionned)
- 2 server for gateway
- 1 server for admin

All servers working on Debian 8 and have a domain name.

- files-01.example.com files-02.example.com files-03.example.com
- files-rgw-01.example.com fles-rgw-02.example.com
- files-admin.example.com

### My target
- Have a cluster distributed on 3 servers with on each server 1 disk for system and 1 monitor and one OSD for each not used disk
- Have a loss capacity of 2 drives without loss of data
- Have 2 gateway in active-active mode to access cluster data
 
## Installation

### On the Admin server

#### Preparation 
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

#### Monitors installation

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
#### OSD Installation 
- Delete partition table of the not used disks in storage servers (in my case /dev/sdb and /dev/sdc on each server)
```
ceph-deploy disk zap files-01:sdb files-01:sdc files-02:sdb files-02:sdc files-03:sdb files-03:sdc
```
- Create the osd 
```
ceph-deploy osd create files-01:sdb files-01:sdc files-02:sdb files-02:sdc files-03:sdb files-03:sdc
```
#### Gateway Installation
- Deploy ceph and radosgw on Gateway server
```
ceph-deploy install --rgw --release luminous files-rgw-01 files-rgw-02
```
- Deploy admin keys on Gateway server
```
ceph-deploy admin files-rgw-01 files-rgw-02
```
- Create the gateway
```
ceph-deploy rgw create files-rgw-01 files-rgw-02
``` 
- Normally, the gateway are running on 7480 port, you can change this port.
- To change the default port add these line in your ~/ceph-cluster/ceph.conf (here i change the default port to 80)
```
[client.rgw.files-rgw-01]
rgw_frontends = "civetweb port=80"

[client.rgw.files-rgw-02]
rgw_frontends = "civetweb port=80"
```
- And push the configuration to your gateway server
```
ceph-deploy --overwrite-conf config push files-rgw-01 files-rgw-02
```
- To apply this change you must restart manually the service on each gateway server with this command
```
systemctl restart ceph-radosgw.service
```
### On the files servers
#### Erasure Code Profile creation
- Create the erasure code profile named data with the data replication on 4 disks and a loss capacity of 2 disks
```
ceph osd erasure-code-profile set data k=4 m=2 crush-failure-domain=osd
```
- However, in my configuration i noticed an error, the crush rule are not perfectly configured automatically with the erasure profile, this cause some errors in pool creation.
- If you meet a "creating+incomplete" error when you creating a pool you must configured the crush map min_size parameters by the value of numbers of disk loss capacity (in my case 2).
- For this, export your crushmap in a file
```
ceph osd getcrushmap > crush.map
```
- Decompile the crushmap
```
crushtool --decompile crush.map > crush.txt
```
- On the crush.txt, in your rule section matching with the name of erasure profile, check the type in "chooseleaf indep" line is "osd"
- Change the min_size to the value of numbers of disk loss capacity (always 2 in my case ^^)
- My ruleset looks like this:
```
rule data {
        ruleset 1
        type 3
        min_size 2
        max_size 6
        step set_chooseleaf_tries 5
        step set_choose_tries 100
        step take default
        step chooseleaf indep 0 type osd
        step emit
}
```

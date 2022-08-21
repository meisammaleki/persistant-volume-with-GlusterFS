# persistant-volume-with-GlusterFS
# persistant-volume-with-GlusterFS
## https://docs.gluster.org/en/latest/Install-Guide/Install/#set-up-a-gluster-volume
## install GlusterFS on ubuntu 20.04
## three servers with running ubuntu 20.04
## set hostname ( gluster0 , gluster1 , gluster2)
## all configuration will be dploy on each three server
## configure /etc/hosts :

192.168.10.10 gluster0.example.com gluster0
192.168.10.11 gluster1.example.com gluster1
192.168.10.12 gluster2.example.com gluster2

## setup software Sources :
apt update
apt install software-properties-common
add-apt-repository ppa:gluster/glusterfs-7
apt update

## install glusterfs-server
apt install glusterfs-server
systemctl status glusterd.service
systemctl start glusterd.service
systemctl enable glusterd.service

## The Gluster daemon uses port 24007, so you’ll need to allow each node access to that port through the firewall of each other node in your storage pool. To do so, run the following command :
## run the following command on gluster0 and the on gluster1 and gluster2 (on each server allow other servers ip)
ufw allow from gluster1_ip_address to any port 24007
ufw allow from gluster2_ip_address to any port 24007

## Configure the trusted pool
## you’ll need to run the "gluster peer probe" command on one of your server. It doesn’t matter which node you use (We run on gluster0) :

gluster peer probe cluster1
gluster peer probe cluster2
gluster peer status

## Partition the disk
fdisk /dev/sdb
mkfs.xfs -i size=512 /dev/sdb1
echo "/dev/sdb1 /export/sdb1 xfs defaults 0 0"  >> /etc/fstab
mkdir -p /export/sdb1 && mount -a && mkdir -p /export/sdb1/brick

## Set up a Gluster volume :
gluster volume create volume1 replica 3 gluster0.example.com:/gluster-storage gluster1.example.com:/gluster-storage gluster2.example.com:/gluster-storage force

## start volume :
gluster volume start volume1
gluster volume status
gluster volume info

## make a directory on gluster0 (as main gluster node) and mount volume1 to that :
mkdir -p /mnt/glusterfs/volume1
mount -t glusterfs 192.168.10.10:/volume1 /mnt/glusterfs/volume1

## now we must install glusterfs-client on worker node on kubernetes cluster :
## add repository and update
apt install glusterfs-client

# at the end deploy an nginx web server with persistant volume 
kubectl create -f gluster-endpoints.yaml
kubectl create -f gluster-pv.yaml
kubectl create -f gluster-claim.yaml
kubectl create -f pod.yaml




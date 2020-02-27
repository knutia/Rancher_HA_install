# Rancher_HA_install
Use 4 VMs to create a Rancher_HA cluster

# Setup / Preporation
1. 4 VMs Ubuntu 18.04.1.0, 1 Loadbalanser, 3 master/worker.
2. Static IPs on individual VMs
3. /etc/hosts hosts file includes name to IP mappings for VMs or DNS
4. Swap is disabled
5. Take snapshots prior to installations, this way you can install and revert to snapshot if needed

# REPEAT FOR ALL 7 VMs

1. Disable swap, swapoff then edit your fstab removing any entry for swap partitions. You can recover the space with fdisk. You may want to reboot to ensure your config is ok.
~~~~
sudo swapoff -a
sudo vi /etc/fstab
~~~~

2. Update the package list and use apt-cache to inspect versions available in the repository
~~~~
sudo apt-get update
sudo apt-get upgrade 
~~~~

# Create Loadbalancer

1. Create nginx config file
~~~~
mkdir /etc/nginx
wget https://raw.githubusercontent.com/knutia/Kubernetes_HA_install/master/nginx.conf
mv ./nginx.conf /etc/nginx/nginx.conf
sudo vi /etc/nginx/nginx.conf
~~~~
Replase <IP/DNSNAME_MASTER_1>, <IP/DNSNAME_MASTER_2> and <IP/DNSNAME_MASTER_3> whit the real ip or FQDN for the 3 masters.

2. Start nginx server using config file
~~~~
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -p 6443:6443 -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.14
~~~~

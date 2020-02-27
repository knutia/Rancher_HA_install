# Rancher_HA_install
Use 4 VMs to create a Rancher_HA cluster

                                                        +------------------+
                                                        |                  |
                                                        |  c3-rancher-ha-1 |
                                        (:80 & :443)  ---   192.168.2.101  |
                                                -----/  |                  |
     +-----------------+                   ----/        +------------------+
     |                 |             -----/             +------------------+
     |                 |        ----/                   |                  |
     |c3-rancher-ha-lb |  -----/        (:80 & :443)    |  c3-rancher-ha-2 |
     |  192.168.2.100  --\-------------------------------   192.168.2.102  |
     |                 |  -----\                        |                  |
     |                 |        ----\                   +------------------+
     |                 |             -----\             +------------------+
     +-----------------+                   ----\        |                  |
                                                -----\  |  c3-rancher-ha-3 |
                                       (:80 & :443)   ---   192.168.2.103  |
                                                        |                  |
                                                        +------------------+

# Setup / Preporation
1. 4 VMs Ubuntu 18.04.1.0, 1 Loadbalanser, 3 master/worker.
2. Static IPs on individual VMs
3. /etc/hosts hosts file includes name to IP mappings for VMs or DNS
4. Swap is disabled
5. Take snapshots prior to installations, this way you can install and revert to snapshot if needed

# Repeat For All 4 VMs

1. Add all nodes to the /etc/hosts hosts file includes name to IP mappings for VMs or DNS
~~~
sudo bash -c 'array=("192.168.2.100 c3-rancher-ha-lb" "192.168.2.101 c3-rancher-ha-1" "192.168.2.102 c3-rancher-ha-2" "192.168.2.103 c3-rancher-ha-3") && for ix in ${!array[*]}; do printf "%s\n" "${array[$ix]}">>/etc/hosts; done'
~~~

2. Disable swap, swapoff then edit your fstab removing any entry for swap partitions. You can recover the space with fdisk. You may want to reboot to ensure your config is ok.
~~~~
sudo swapoff -a
sudo vi /etc/fstab
~~~~

3. Update the package list and use apt-cache to inspect versions available in the repository
~~~~
sudo apt-get update && sudo apt-get upgrade -y && sudo apt autoremove -y
~~~~

# Create Loadbalancer

1. Install the required packages, if needed we can request a specific version
~~~~
sudo apt-get install -y docker.io
~~~~

2. Ensure both are set to start when the system starts up.
~~~~
sudo systemctl enable docker.service
~~~~

3. Create nginx config file
~~~~
mkdir /etc/nginx
wget -O /etc/nginx/nginx.conf https://raw.githubusercontent.com/knutia/Kubernetes_HA_install/master/nginx.conf
~~~~
Replase <IP_DNSNAME_HA_1>, <IP_DNSNAME_HA_2> and <IP_DNSNAME_HA_3> whit the real ip or FQDN for the 3 master nodes in the /etc/nginx/nginx.conf file.
Eksample:
~~~
sed -i 's/<IP_DNSNAME_HA_1>/192.168.2.101/g' /etc/nginx/nginx.conf
sed -i 's/<IP_DNSNAME_HA_2>/192.168.2.102/g' /etc/nginx/nginx.conf
sed -i 's/<IP_DNSNAME_HA_3>/192.168.2.103/g' /etc/nginx/nginx.conf
~~~


4. Start nginx server using config file
~~~~
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 -p 6443:6443 -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf nginx:1.14
~~~~

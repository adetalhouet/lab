
### Setup

~~~
dnf install dhcp-server bind haproxy -y
systemctl start dhcpd
systemctl enable dhcpd
systemctl start named
systemctl enable named
systemctl start haproxy
systemctl enable haproxy
dnf install -y bind-utils libguestfs-tools cloud-init libvirt
dnf module install virt -y
dnf install virt-install -y
systemctl enable libvirtd --now
sysctl -w net.ipv4.ip_forward=1
~~~


##### Hugepages
~~~
vi /etc/sysctl.conf
vm.nr_hugepages = 52428
vm.hugetlb_shm_group = 36
~~~

~~~
vi /etc/security/limits.conf
soft memlock 107374200
hard memlock 107374200
~~~

####

##### NFS
~~~
dnf install nfs-utils -y
mkdir -p /mnt/nfs_shares
chown -R nobody: /mnt/nfs_shares
chmod -R 777 /mnt/nfs_shares

systemctl start nfs-server.service
systemctl enable nfs-server.service

firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --permanent --add-service=mountd
firewall-cmd --reload
~~~

###### on the server
~~~
echo "/mnt/nfs_shares 136.243.41.43(rw,sync,no_all_squash,root_squash)" >> /etc/exports
exportfs -arv
exportfs -s # check the export list
systemctl restart nfs-utils.service
~~~

###### on the client
~~~
mount -t nfs 148.251.12.17:/mnt/nfs_shares /mnt/nfs_shares
echo "148.251.12.17:/mnt/nfs_shares /mnt/nfs_shares   nfs    auto  0  0" >> /etc/fstab
~~~

##### vSwitch
~~~
vi /etc/sysconfig/network-scripts/ifcfg-enp35s0.4000
DEVICE=enp35s0.4000
BOOTPROTO=none
ONBOOT=yes
VLAN=yes
BRIDGE=br-int

vi /etc/sysconfig/network-scripts/ifcfg-br-int
DEVICE=br-int
TYPE=Bridge
BOOTPROTO=static
ONBOOT=yes
IPADDR=10.64.1.3
NETWORK=10.64.1.0
PREFIX=24
VLAN=yes
DNS1=127.0.0.1
~~~

ip link set mtu 1400 enp4s0.4000

##### IP forward
~~~
iptables -t nat -L --line-numbers -n
iptables --flush
iptables --table nat --flush
iptables --delete-chain
iptables --table nat --delete-chain
iptables --table nat --append POSTROUTING --out-interface br0 -j MASQUERADE
iptables -A FORWARD --in-interface br-int -j ACCEPT

yum install iptables-services -y
systemctl enable iptables
service iptables save

~~~

##### DHCP
~~~
dnf install dhcp-server -y

vi /etc/dhcp/dhcpd.conf

systemctl start dhcpd
systemctl enable dhcpd
~~~
##### DNS
~~~
dnf install bind -y

systemctl start named
systemctl enable named
~~~
##### Loadbalancer
~~~
dnf install haproxy -y

vi /etc/haproxy/haproxy.conf

systemctl start haproxy
systemctl enable haproxy
~~~
##### Firewall
~~~
firewall-cmd --add-port=443/tcp --permanent --zone=libvirt
firewall-cmd --add-port=6443/tcp --permanent --zone=libvirt
firewall-cmd --add-port=80/tcp --permanent --zone=libvirt
firewall-cmd --add-port=22623/tcp --permanent --zone=libvirt

firewall-cmd --add-port=443/tcp --permanent --zone=public
firewall-cmd --add-port=6443/tcp --permanent --zone=public
firewall-cmd --add-port=80/tcp --permanent --zone=public
firewall-cmd --add-port=22623/tcp --permanent --zone=public

firewall-cmd --permanent --add-masquerade
firewall-cmd --reload
~~~
##### Libvirt

-- OCS disk
~~~
qemu-img create -f raw /var/lib/libvirt/images/worker1-ocs.qcow2 350G
virsh attach-disk worker4 --source /var/lib/libvirt/images/worker4-ocs.qcow2 --target vdb --cache none
~~~
-- install
~~~
dnf install -y bind-utils libguestfs-tools cloud-init libvirt
dnf module install virt -y
dnf install virt-install -y
systemctl enable libvirtd --now
~~~
-- network
~~~
virsh net-define net.xml
virsh net-start br-int
virsh net-autostart br-int

~~~
-- Add interface for CNV
~~~
virsh attach-interface --domain worker6 --type network --source default --model virtio --config --live
~~~

-- VM
~~~

qemu-img create -f qcow2 /var/lib/libvirt/images/montreal-node1.qcow2 120G


virsh destroy saskatoon-node1
virsh destroy saskatoon-node2
virsh destroy saskatoon-node3
virsh undefine saskatoon-node1
virsh undefine saskatoon-node2
virsh undefine saskatoon-node3
rm -rf /var/lib/libvirt/images/saskatoon*.qcow2
virsh define node1.xml
virsh define node3.xml
virsh define node2.xml
qemu-img create -f qcow2 /var/lib/libvirt/images/saskatoon-node1.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/saskatoon-node2.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/saskatoon-node3.qcow2 120G
virsh start saskatoon-node1
virsh start saskatoon-node2
virsh start saskatoon-node3


virsh destroy regina-node1
virsh destroy regina-node2
virsh destroy regina-node3
virsh undefine regina-node1
virsh undefine regina-node2
virsh undefine regina-node3
rm -rf /var/lib/libvirt/images/regina*.qcow2
virsh define node1.xml
virsh define node3.xml
virsh define node2.xml
qemu-img create -f qcow2 /var/lib/libvirt/images/regina-node1.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/regina-node2.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/regina-node3.qcow2 120G
virsh start regina-node1
virsh start regina-node2
virsh start regina-node3

virsh destroy master1
virsh destroy master2
virsh destroy master3
virsh undefine master1
virsh undefine master2
virsh undefine master3
rm -rf /var/lib/libvirt/images/master*.qcow2
qemu-img create -f qcow2 /var/lib/libvirt/images/master1.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/master2.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/master3.qcow2 120G
virsh define master1.xml
virsh define master2.xml
virsh define master3.xml
virsh start master1
virsh start master2
virsh start master3

virsh destroy worker1
virsh destroy worker3
virsh destroy worker2
virsh destroy worker4
virsh destroy worker5
virsh undefine worker1
virsh undefine worker3
virsh undefine worker2
virsh undefine worker4
virsh undefine worker5
rm -rf /var/lib/libvirt/images/worker*.qcow2
qemu-img create -f qcow2 /var/lib/libvirt/images/worker1.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/worker2.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/worker3.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/worker4.qcow2 120G
qemu-img create -f qcow2 /var/lib/libvirt/images/worker5.qcow2 120G
qemu-img create -f raw /var/lib/libvirt/images/worker1-ocs.qcow2 150G
qemu-img create -f raw /var/lib/libvirt/images/worker2-ocs.qcow2 150G
qemu-img create -f raw /var/lib/libvirt/images/worker3-ocs.qcow2 150G
qemu-img create -f raw /var/lib/libvirt/images/worker4-ocs.qcow2 150G
qemu-img create -f raw /var/lib/libvirt/images/worker5-ocs.qcow2 150G
virsh define worker5.xml
virsh define worker4.xml
virsh define worker3.xml
virsh define worker2.xml
virsh define worker1.xml
virsh start worker1
virsh start worker2
virsh start worker3
virsh start worker4
virsh start worker5

virsh destroy worker4
virsh undefine worker4
rm -rf /var/lib/libvirt/images/worker4.qcow2
qemu-img create -f qcow2 /var/lib/libvirt/images/worker4.qcow2 200G
virsh define worker4.xml
virsh start worker4

virsh destroy worker5
virsh undefine worker5
rm -rf /var/lib/libvirt/images/worker5.qcow2
qemu-img create -f qcow2 /var/lib/libvirt/images/worker5.qcow2 200G
virsh define worker5.xml
virsh start worker5

virsh destroy worker6
virsh undefine worker6
rm -rf /var/lib/libvirt/images/worker6.qcow2
qemu-img create -f qcow2 /var/lib/libvirt/images/worker6.qcow2 200G
virsh define worker6.yaml
virsh start worker6

~~~
##### Backup /etc/resolv.conf
~~~
nameserver 213.133.98.98
nameserver 213.133.99.99
nameserver 213.133.100.100
nameserver 2a01:4f8:0:1::add:9898
nameserver 2a01:4f8:0:1::add:9999
nameserver 2a01:4f8:0:1::add:1010
~~~
qemu-img create -f qcow2 /var/lib/libvirt/images/pfsense.qcwo2 100G

virt-install --virt-type kvm --name pfsense --ram 8162 --vcpus 8 \
--cdrom=/var/lib/libvirt/boot/pfSense-CE-2.5.2-RELEASE-amd64.iso \
--disk /var/lib/libvirt/images/pfsense.qcwo2,bus=virtio,size=100,format=qcow2 \
--network default \
--network bridge=br-ext \
--graphics vnc,listen=0.0.0.0 --noautoconsole \
--os-type=linux --os-variant=freebsd10.0
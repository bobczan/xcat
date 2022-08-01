#!/bin/bash

if [ ! -f /tmp/hostname ]; then

echo
echo "Use default host name 'xcatmn.cluster.com'"
echo
echo "  y  "
echo
echo "  n  "

read k
case $k in
  y) echo "xcatmn.cluster.com" > /etc/hostname; touch /tmp/hostname;;
  n) read -p "Enter host name: " HOSTNAME1; echo "$HOSTNAME1" > /etc/hostname; touch /tmp/hostname;;
  *) echo "invalid option";exit;;
esac;

fi

echo
echo Hostname changed:
echo
cat /etc/hostname
echo
sleep 5
echo

if [ ! -f /tmp/yumupdate ]; then

echo "Update os?"
echo "System will reboot afer update"
echo
echo "  y   "
echo
echo "  n   "
echo
read m
case $m in
  y) yum update -y; touch /tmp/yumupdate; echo "run me again after reboot"; reboot;;
  n) ;;
  *) ;;
esac;
fi

if [ ! -f /usr/bin/wget ]; then
echo "wget not installad, installation in progress"
yum install wget -y >>/dev/null
fi

myarray=(`find /tmp/ -maxdepth 1 -name "*.iso"`)
if [ ${#myarray[@]} -gt 0 ]; then 
    echo true 
else 

echo
    echo Centos-DVD iso doesnt exist in /tmp
echo
echo "  y       Start Dowloading ISO"
echo
echo "  n       Copy manualy"


read m
case $m in
    y) wget http://mirror.bytemark.co.uk/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-DVD-2009.iso -O /tmp/CentOS-7-x86_64-DVD-2009.iso ;; 
	#http://centos.serverspace.co.uk/centos/7.7.1908/isos/x86_64/CentOS-7-x86_64-DVD-1908.iso -O /tmp/CentOS-7-x86_64-DVD-1908.iso ;;
    n) echo "Run script again once iso copied";exit;;
    *) echo "invalid option";exit;;
esac;

fi

echo
echo "Downloading go-xcat"
echo


if [ ! -f /tmp/go-xcat ]; then

wget https://raw.githubusercontent.com/xcat2/xcat-core/master/xCAT-server/share/xcat/tools/go-xcat -O - >/tmp/go-xcat

fi

chmod +x /tmp/go-xcat

if [ ! -f /opt/xcat/sbin/xcatd ]; then

/tmp/go-xcat install -y

sleep 10

echo "run me again after reboot"

reboot

fi

echo "source /etc/profile.d/xcat.sh" >> /root/.bashrc

source /root/.bashrc



echo
INTERFACES=( $(ip -o link | awk -F : '!/LOOPBACK/ {print $2}') )

# Print choices
echo "Available interface"
for i in "${!INTERFACES[@]}"; do
  printf "%s) %s\n" "$i" "${INTERFACES[$i]}"
done

echo
echo "Choice external connection"
echo
echo
read -p "Enter choice: " CHOICE
echo
echo
echo "Choice internal connection"
echo
echo
read -p "Enter choice: " CHOICE2
echo
echo "External ${INTERFACES[$CHOICE]}"
echo
echo "Internal ${INTERFACES[$CHOICE2]}"
echo


perl -pi -e "s/BOOTPROTO=dhcp/BOOTPROTO=static/" /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}";

perl -pi -e "s/ONBOOT=no/ONBOOT=yes/" /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}";

perl -pi -e '$line = 1 if /IPADDR=192.168.250.250/; print "IPADDR=192.168.250.250\n" if !$line&&eof' /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}";

perl -pi -e '$line = 1 if /NETMASK=255.255.0.0/; print "NETMASK=255.255.0.0\n" if !$line&&eof' /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}";

perl -pi -e '$line = 1 if /PREFIX=16/; print "PREFIX=16\n" if !$line&&eof' /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}";

perl -pi -e '$line = 1 if /home/; print "/home *(rw,no_root_squash,sync,no_subtree_check)\n" if !$line&&eof' /etc/exports;

perl -pi -e '$line = 1 if /install/; print "/install *(rw,no_root_squash,sync,no_subtree_check)\n" if !$line&&eof' /etc/exports;

#cat /etc/sysconfig/network-scripts/ifcfg-"${INTERFACES[$CHOICE2]}"

ifup "${INTERFACES[$CHOICE2]}"

read -p "Set root password for compute image: " PA55WORD; chtab key=system passwd.username=root passwd.password="$PA55WORD";

echo
echo "Setting up DHCP interface ${INTERFACES[$CHOICE2]}"
echo

chdef -t site clustersite dhcpinterfaces=${INTERFACES[$CHOICE2]}

chdef -t site clustersite master=192.168.250.250
chdef -t site clustersite nameservers=192.168.250.250
chdef -t site clustersite forwarders=8.8.8.8

sleep 5

chdef -t site clustersite domain=$(hostname -d)

lsdef -t site clustersite

makenetworks -n

makedhcp -n
echo
echo "Print network information"
echo
lsdef -t network 192_168_0_0-255_255_0_0

echo
echo "Using DVD iso to make images"
echo
copycds /tmp/CentOS*.iso

echo
echo "List of available images"
echo
lsdef -t osimage

sleep 10

genimage centos7.9-x86_64-netboot-compute

sleep 10

genimage centos7.9-x86_64-install-compute

echo
echo "Enabling repo in the image"
echo

mv /install/netboot/centos7.9/x86_64/compute/rootimg/etc/yum.repos.d/CentOS-Base.repo /install/netboot/centos7.9/x86_64/compute/rootimg/etc/yum.repos.d/CentOS-Base.repo.backup

cp /etc/yum.repos.d/CentOS-Base.repo /install/netboot/centos7.9/x86_64/compute/rootimg/etc/yum.repos.d/

perl -pi -e '$line = 1 if /"CHROOT="/; print "CHROOT=/install/netboot/centos7.9/x86_64/compute/rootimg/\n" if !$line&&eof' /root/.bashrc;

source /root/.bashrc

echo
echo "Installing basic packages to stateless image"
echo

yum -y --installroot=$CHROOT install ipmitool yum epel-release htop glances screen vim

echo
echo "To install something else run 'xcat-image-install'"
echo

echo 'alias xcat-image-install="yum -y --installroot=$CHROOT install"' >> /root/.bash_profile

source /root/.bash_profile

echo
echo "Adding shared home location to the image"
echo

perl -pi -e '$line = 1 if /home/; print "192.168.250.250:/home /home nfs defaults 0 0\n" if !$line&&eof' $CHROOT/etc/fstab;

echo adding files to sync

mkdir -p /install/custom/netboot/

cat > /install/custom/netboot/compute.synclist << 'EOF'

/etc/passwd -> /etc/passwd
/etc/shadow -> /etc/shadow
/etc/group -> /etc/group
/etc/hosts -> /etc/hosts

EOF

#lsdef -t osimage

chdef -t osimage -o centos7.9-x86_64-netboot-compute synclists="/install/custom/netboot/compute.synclist"

mkdir -p /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh

#chmod 700 /install/netboot/centos7.9/x86_64/compute/rootimg/.ssh
chmod 700 /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh

cat ~/.ssh/id_rsa.pub > /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh/authorized_keys

chmod 600 /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh/authorized_key

echo "vim /install/custom/netboot/compute.synclist;" >> /usr/local/bin/sync-node

echo "updatenode cn1 -F" >> /usr/local/bin/sync-node

echo "alias sync-node=/usr/local/bin/sync-node" >> /root/.bash_profile

chmod +x /usr/local/bin/sync-node

echo
echo "Building stateless image"
echo

packimage centos7.9-x86_64-netboot-compute

echo
echo "Building installation image"
	echo

packimage centos7.9-x86_64-install-compute


systemctl status firewalld
systemctl start firewalld
systemctl enable firewalld
echo
echo Setting up firewall
echo
firewall-cmd --zone=external --add-interface=${INTERFACES[$CHOICE1]} --permanent
firewall-cmd --zone=internal --add-interface=${INTERFACES[$CHOICE2]} --permanent
firewall-cmd --zone=external --add-masquerade --permanent
systemctl restart firewalld
firewall-cmd --permanent --direct --passthrough ipv4 -t nat -I POSTROUTING -o ${INTERFACES[$CHOICE1]} -j MASQUERADE
firewall-cmd --permanent --zone=internal --add-service=dhcp
firewall-cmd --permanent --zone=internal --add-service=tftp
firewall-cmd --permanent --zone=internal --add-service=dns
firewall-cmd --permanent --zone=internal --add-service=http
firewall-cmd --permanent --zone=internal --add-service=nfs
firewall-cmd --permanent --zone=internal --add-service=ssh
firewall-cmd --permanent --zone=internal --add-service=mountd
firewall-cmd --permanent --zone=internal --add-service=rpc-bind
firewall-cmd --complete-reload
firewall-cmd --list-all-zones
firewall-cmd --permanent --zone=internal --add-port=6817/tcp
firewall-cmd --permanent --zone=internal --add-port=6817/udp
firewall-cmd --permanent --zone=internal --add-port=6818/tcp
firewall-cmd --permanent --zone=internal --add-port=6818/udp
firewall-cmd --permanent --zone=external --add-service=http
firewall-cmd --permanent --zone=external --add-service=https
firewall-cmd --permanent --zone=internal --add-port=8660/tcp
firewall-cmd --permanent --zone=internal --add-port=8661/tcp
firewall-cmd --permanent --zone=internal --add-port=8662/tcp
firewall-cmd --permanent --zone=internal --add-port=8663/tcp
firewall-cmd --permanent --zone=internal --add-port=8660/udp
firewall-cmd --permanent --zone=internal --add-port=8661/udp
firewall-cmd --permanent --zone=internal --add-port=8662/udp
firewall-cmd --permanent --zone=internal --add-port=8663/udp
firewall-cmd --permanent --zone=internal --add-port=8651/udp
firewall-cmd --permanent --zone=internal --add-port=8651/tcp
firewall-cmd --zone trusted --change-interface="${INTERFACES[$CHOICE2]}"
systemctl restart firewalld

echo
echo Creating DNS
echo

makedns -n

echo
echo Checking health of xCat
echo

xcatprobe xcatmn -i ${INTERFACES[$CHOICE2]}

echo "source /etc/profile.d/xcat.sh" >> /root/.bashrc

echo
echo "Available os images"
echo
lsdef -t osimage

cat > /usr/local/bin/add-node << 'EOF' 
#!/bin/bash
read -p "Node Name : " NODENAME;
read -p "Mac address in format aa:bb:cc:dd:ee:ff : " MAC;
read -p "IP ADDRESS 192.168.0.xxx : " IPADDR;
mkdef -t node $NODENAME --template x86_64-template ip=192.168.0.$IPADDR mac=$MAC bmc=192.168.100.$IPADDR bmcusername=ADMIN bmcpassword=ADMIN
echo
echo New node info
echo
lsdef $NODENAME;
echo
echo Node list
echo
nodels
makehosts $NODENAME;
nodeset $NODENAME osimage=centos7.9-x86_64-netboot-compute
EOF

chmod +x /usr/local/bin/add-node

echo "alias add-node=/usr/local/bin/add-node" >> /root/.bash_profile

echo
echo "source /root/.bash_profile"
echo

echo
echo "To add Compute Node run 'add-node'"
echo
echo
echo "To install packages in to the stateless image run: 'xcat-image-install'"
echo
echo "Image info"
echo

lsdef -t osimage centos7.9-x86_64-netboot-compute
systemctl disable --now NetworkManager

#nodeset cn1 osimage
#chdef -t node -o cn1 netboot=yaboot
#nodeset cn12 osimage=centos7.9-x86_64-netboot-compute
#chdef -t node -o cn12 -p addkcmdline="bootdev=ib0 ksdevice=ib0 net.ifnames=0 biosdevname=0 rd.neednet=1 rd.bootif=0 rd.driver.pre=mlx5_ib,mlx4_ib,ib_ipoib ip=ib0:static rd.net.dhcp.retry=10 rd.net.timeout.iflink=60 rd.net.timeout.ifup=80 rd.net.timeout.carrier=80"

#tabedit networks
#chdef -t site dhcpinterfaces="eth1,eth3"




#####test#####
#
#
#mkdir -p /install/custom/netboot/
#
#cat > /install/custom/netboot/compute.synclist << 'EOF'
#
#/etc/passwd -> /etc/passwd
#/etc/shadow -> /etc/shadow
#/etc/group -> /etc/group
#/etc/hosts -> /ets/hosts
#
#EOF
#
##lsdef -t osimage
#
#chdef -t osimage -o centos7.9-x86_64-netboot-compute synclists="/install/custom/netboot/compute.synclist"
#
#mkdir /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh
#
#chmod 700 /install/netboot/centos7.9/x86_64/compute/rootimg/.ssh
#
#cat ~/.ssh/id_rsa.pub > /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh/authorized_keys
#
#chmod 600 /install/netboot/centos7.9/x86_64/compute/rootimg/root/.ssh/authorized_key

# Lustre-2.10.4
lustre是一个分布式文件系统，lustre2.10.4兼容多版本操作系统(ubuntu16、CentOS6、CentOS7),以下是学习心得，如有不对之处，请指正，后续会陆续增加自己的学习心得！

系统环境

CentOS Linux release 7.3.1611	存储服务端，客户端	192.168.179.140 node03	内存8GB，4核，50+20GB存储

Ubuntu 16.04.3	客户端	192.168.179.141 node01	内存4GB，4核，20GB存储

CentOS Linux release 7.3.1611	客户端	192.168.179.134 node02	内存8GB，4核，50+20GB存储

下载地址：https://downloads.whamcloud.com/public/

1.时间同步

(1)ntpd,过程简单，略

2.无密访问配置

(1)ssh-keygen,过程简单，略

3.关闭防火墙、selinux


4.安装部署

安装lustre服务端

yum install \
asciidoc audit-libs-devel automake bc \
binutils-devel bison device-mapper-devel elfutils-devel \
elfutils-libelf-devel expect flex gcc gcc-c++ git \
glib2 glib2-devel hmaccalc keyutils-libs-devel krb5-devel ksh \
libattr-devel libblkid-devel libselinux-devel libtool \
libuuid-devel libyaml-devel lsscsi make ncurses-devel \
net-snmp-devel net-tools newt-devel numactl-devel \
parted patchutils pciutils-devel perl-ExtUtils-Embed \
pesign python-devel redhat-rpm-config rpm-build systemd-devel \
tcl tcl-devel tk tk-devel wget xmlto yum-utils zlib-devel


#1.安装lustre依赖的系统内核

yum install \
kernel-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-devel-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-headers-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
dracut-033-535.el7.x86_64.rpm dracut-config-rescue-033-535.el7.x86_64.rpm	\
dracut-network-033-535.el7.x86_64.rpm linux-firmware-20180220-62.2.git6d51311.el7_5.noarch.rpm \
kexec-tools-2.0.15-13.el7.x86_64.rpm		
#
yum install kernel-tools-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-tools-libs-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-tools-libs-devel-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-tools-debuginfo-3.10.0-862.2.3.el7_lustre.x86_64.rpm \
kernel-debuginfo-common-x86_64-3.10.0-862.2.3.el7_lustre.x86_64.rpm 

#重启机器

shutdown -r now


#2.安装e2fsprogs	

yum install e2fsprogs-1.42.13.wc6-7.el7.x86_64.rpm \
e2fsprogs-debuginfo-1.42.13.wc6-7.el7.x86_64.rpm \
e2fsprogs-devel-1.42.13.wc6-7.el7.x86_64.rpm \
e2fsprogs-libs-1.42.13.wc6-7.el7.x86_64.rpm \
e2fsprogs-static-1.42.13.wc6-7.el7.x86_64.rpm \
libcom_err-1.42.13.wc6-7.el7.x86_64.rpm \
libcom_err-devel-1.42.13.wc6-7.el7.x86_64.rpm \
libss-1.42.13.wc6-7.el7.x86_64.rpm libss-devel-1.42.13.wc6-7.el7.x86_64.rpm

#3.安装zfs
yum -y install http://download.zfsonlinux.org/epel/zfs-release.el7_4.noarch.rpm
yum install zfs kmod-lustre-osd-zfs-2.10.4-1.el7.x86_64.rpm -y
yum install spl-0.7.9-1.el7.x86_64.rpm spl-debuginfo-0.7.9-1.el7.x86_64.rpm -y
yum install zfs-debuginfo zfs-dracut zfs-test -y
#4.安装lustre 	
yum install kmod-lustre-2.10.4-1.el7.x86_64.rpm \
kmod-lustre-osd-ldiskfs-2.10.4-1.el7.x86_64.rpm \
lustre-osd-ldiskfs-mount-2.10.4-1.el7.x86_64.rpm -y
#		
yum install \
lustre-2.10.4-1.el7.x86_64.rpm \
lustre-osd-ldiskfs-mount-2.10.4-1.el7.x86_64.rpm \
lustre-resource-agents-2.10.4-1.el7.x86_64.rpm \
lustre-iokit-2.10.4-1.el7.x86_64.rpm \
lustre-tests-2.10.4-1.el7.x86_64.rpm \
kmod-lustre-tests-2.10.4-1.el7.x86_64.rpm -y
#
yum install \
lustre-all-dkms-2.10.4-1.el7.noarch.rpm \
lustre-osd-zfs-mount-2.10.4-1.el7.x86_64.rpm \
zfs-dkms-0.7.9-1.el7.noarch.rpm \
spl-dkms-0.7.9-1.el7.noarch.rpm \
dkms-2.6.1-1.el7.noarch.rpm \
libzfs2-0.7.9-1.el7.x86_64.rpm \
libzpool2-0.7.9-1.el7.x86_64.rpm \
libuutil1-0.7.9-1.el7.x86_64.rpm \
libnvpair1-0.7.9-1.el7.x86_64.rpm \
libzfs2-devel-0.7.9-1.el7.x86_64.rpm -y

#将lustre模块导入内核

modprobe zfs
modprobe lustre		
modprobe lnet
modprobe ldiskfs	

#查看导入是否成功

lsmod 

#硬盘分区以及格式化

##分区

parted -s /dev/sdb "mkpart primary 0% 20%"
parted -s /dev/sdb "mkpart primary 20% 40%"
parted -s /dev/sdb "mkpart primary 40% 60%"
parted -s /dev/sdb "mkpart primary 60% 80%"
parted -s /dev/sdb "mkpart primary 80% 100%"

##格式化

mkfs.xfs /dev/sdb1 -f
mkfs.xfs /dev/sdb2 -f
mkfs.xfs /dev/sdb3 -f

#lustre配置

##MDS

mkfs.lustre --reformat --mgs --backfstype=zfs --fsname=lustre --device-size=1048576 lustre-mgs/mgs /dev/sdb1

##MDT

mkfs.lustre --reformat --mdt --backfstype=zfs --fsname=lustre --index=0 --mgsnode=node03@tcp --device-size=1048576 lustre-mdt0/mdt0 /dev/sdb2

##OST

mkfs.lustre --reformat --ost --backfstype=zfs --fsname=lustre --index=0 --mgsnode=node03@tcp --device-size=1048576 lustre-ost0/ost0 /dev/sdb3

##配置lnet

#/etc/modprobe.d/lustre.conf 
options lnet networks="tcp0(ens33)"

##配置ldev

#/etc/ldev.conf

node03 - mgs     zfs:lustre-mgs/mgs
node03 - mdt0    zfs:lustre-mdt0/mdt0
node03 - ost0    zfs:lustre-ost0/ost0

#启动lustre服务

service lustre start

#挂载

mount -t lustre node03:/lustre /mnt/

客户端lustre安装

#CentOS7同服务端一样的安装

#挂载

mount -t lustre node03:/lustre /mnt/

#Ubuntu

dpkg -i \
lustre-client-modules-4.4.0-116-generic_2.10.4-1_amd64.deb \
lustre-dev_2.10.4-1_amd64.deb

#挂载

mount -t lustre node03:/lustre /mnt/

#

附件
http://wiki.lustre.org/Installing_the_Lustre_Software
http://wiki.lustre.org/Lustre_with_ZFS_Install
http://cdn.opensfs.org/wp-content/uploads/2017/06/Wed04-BlagodarenkoArtem-Scaling20ldiskfs20for20the20future.20Again.20LUG202017-2.pdf
http://wiki.lustre.org/Integrated_Manager_for_Lustre
https://github.com/whamcloud/integrated-manager-for-lustre/
http://wiki.lustre.org/Lustre_Monitoring_Tool
https://github.com/LLNL/lmt


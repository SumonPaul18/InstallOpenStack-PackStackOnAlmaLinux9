
# Install OpenStack-PackStack on AlmaLinux 9

Checking System Informations
####
    hostnamectl
####
    cat /etc/redhat-release
####
    grep -E ' svm | vmx' /proc/cpuinfo
####
    lsmod | grep kvm
####
    lscpu
####
    free -h
####
    lsblk

#### Common Configuration 

    hostnamectl set-hostname cloud
####
    ip a
####
    yum install nano -y

#### Need to Configure Static IP

Check Network Card Info file
####
    ls /etc/sysconfig/network-scripts/

> Note: If OS are Almalinux 9 may be couldn't find NIC file in network-scripts folder. 
#### Solutions for thats problem:

Enable CRB  Repository
####
    dnf config-manager --set-enabled crb

Install Epel and Epel Next on Almalinux 9
####
    dnf install epel-release -y

Working On AlmaLinux 9
####
    dnf install centos-release-openstack-yoga -y

    yum clean all

Install network-scripts package
####
    yum install network-scripts -y

Enable/Start Network Service
####
    systemctl status network
    systemctl start network
    systemctl enable network

    systemctl restart network

Check Network Card Info file
####
    ls /etc/sysconfig/network-scripts/
####
    nano /etc/sysconfig/network-scripts/ifcfg-enp1s0
####

    HWADRR=1c:1b:0d:8b:c6:ba
    NM_CONTROLLED=no
    BOOTPROTO=static
    ONBOOT=yes
    IPADDR=192.168.0.50
    PREFIX=24
    GATEWAY=192.168.0.1
    DNS1=8.8.8.8
    DNS2=8.8.4.4
    DEVICE=enp1s0
####
    nmcli connection up enp1s0
####
    ip a s enp1s0
####
    systemctl restart NetworkManager
####
    ping google.com
####
    cp /etc/sysconfig/network-scripts/ifcfg-enp1s0 /etc/sysconfig/network-scripts/ifcfg-enp1s0.bak
####
    cat /etc/sysconfig/network-scripts/ifcfg-enp1s0

Edit Hosts file:
####
    echo "192.168.0.50 cloud.paulco.xyz cloud" >> /etc/hosts

Check SELinux
####
    getenforce

Disable SELinux
####
    sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config

Disable Firewalld
####
    systemctl disable firewalld
    systemctl stop firewalld

Disable/Stop NetworkManager
####
    systemctl status NetworkManager
    systemctl disable NetworkManager
    systemctl stop NetworkManager

####
    ifup enp0s3

####
    yum autoremove epel-release
####
    yum autoremove openstack-packstack
####
    yum clean all
####
    yum repolist
####
    yum update -y && yum upgrade -y
####
    reboot

#### Install OpenStack-PackStack Repository

Setup CRB Config-Manager
####
    dnf config-manager --set-enabled crb

Now, Install OpenStack-Yoga Version Package
####
    dnf install centos-release-openstack-yoga

#### Install Openstack-Packstack Package

####
    yum install -y openstack-packstack
OR
####
    dnf install -y openstack-packstack
#### Verifying YUM Repolist
    yum repolist
#### Update
    yum update -y
Verifying PackStack Version
#### 
    packstack --version
Verifying PackStack Help or Manual
#### 
    packstack --help

#### Now, Generate Answers file for Install and Post Configured OpenStack PackStack

#### Pre-Installation with Generate Answers file

    packstack --os-neutron-ml2-tenant-network-types=vxlan \
    --os-neutron-ovs-bridge-interfaces=br-ex:enp1s0 \
    --os-neutron-ml2-mechanism-drivers=openvswitch \
    --os-neutron-ml2-type-drivers=vxlan,flat \
    --os-neutron-l2-agent=openvswitch \
    --keystone-admin-passwd=openstack \
    --provision-demo=n \
    --os-heat-install=y \
    --gen-answer-file /root/answers.txt
 
#### Custom Installation After Generate Answers file

    packstack --gen-answer-file /root/answers.txt

Take Copy the answes file

    cp answers.txt copy.answers.txt


#### Important: Edit the file whats we install 

    nano answers.txt

####

    CONFIG_NTP_SERVERS=0.asia.pool.ntp.org,1.asia.pool.ntp.org,2.asia.pool.ntp.org,3.asia.pool.ntp.org

    CONFIG_CONTROLLER_HOST=192.168.10.10

    CONFIG_COMPUTE_HOSTS=192.168.10.10

    CONFIG_NETWORK_HOSTS=192.168.10.10

    CONFIG_STORAGE_HOST=192.168.10.10

    CONFIG_KEYSTONE_ADMIN_PW=openstack

    # Networking 
    #search and change Bridge
    CONFIG_NEUTRON_OVS_BRIDGE_IFACES=br-ex:enp0s3
    CONFIG_NEUTRON_ML2_MECHANISM_DRIVERS=openvswitch
    CONFIG_NEUTRON_L2_AGENT=openvswitch

    # Disable Nagios
    CONFIG_NAGIOS_INSTALL=n

    # Disable the demo provisioning
    CONFIG_PROVISION_DEMO=n

#### Installation OpenStack
Run bellow command without #
####
    packstack --answer-file #/root/answers.txt | tee Openstack-Installation-log.txt

<details> 
<summary> Run, Packstack Command With Or Not WithOut Demo Project </summary>

####
Install OpenStack With Demo

####
    #packstack --allinone
Install OpenStack Without Demo Project
####
    #packstack --allinone --provision-demo=n
</details>  

#### View Installation Log
The installation log file is available at: 
####
    tail -f /var/tmp/packstack/latest/openstack-setup.log
##### OR 
The generated manifests are available at: 
####
    tail -f /var/tmp/packstack/20230119-111007-GkD8Cn/manifests

## Installation Completed Successfully 


## After Installation Complete

#### Access OpenStack from CLI / Horizon Dashboard

    source ~/keystonerc_admin
####
    openstack service list

To access Horizon Dashboard use the URL: 
####   
    http://ServerIPAddress/dashboard.
    http://192.168.0.50/dashboard

#### Login Credentials in Here:

    cat ~/keystonerc_admin


#### Configuration OpenStack

Now Download Cirros image for OpenStack Instance OS
####
    wget http://download.cirros-cloud.net/0.5.1/cirros-0.5.1-x86_64-disk.img

#### Upload-Create Image on OpenStack

    openstack image create --disk-format qcow2 --container-format bare --public --file cirros-0.5.1-x86_64-disk.img  cirros
####
    openstack image list
####
    #openstack image delete cirros


#### Remove or Uninstall OpenStack

If you need to uninstall openstack
####
    #yum -y remove openstack-*

# Finish 

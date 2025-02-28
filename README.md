# RaspberryPi-Nvme_over_TCP

#### 树莓派target端
```
#注册虚拟nvme设备
sudo fallocate -l 10G /root/nvme_disk.img
sudo losetup /dev/loop99 /root/nvme_disk.img
sudo modprobe nvme
sudo nvme connect-all
sudo mkfs.ext4 /dev/nvme1n1
sudo mkdir /mnt/nvme
sudo mount /dev/nvme1n1 /mnt/nvme

sudo ip link set eth0 up
```

```
#配置nvme over TCP
sudo su
sudo modprobe nvmet_tcp
mkdir /sys/kernel/config/nvmet/subsystems/nvme-subsystem-name
cd /sys/kernel/config/nvmet/subsystems
cd nvme-subsystem-name/  # 设置允许所有主机访问
echo 1 > attr_allow_any_host
cd namespaces/  # 直接使用准备申请的NSID作为目录名创建目录
mkdir 10
cd 10
echo "/dev/nvme-fabrics" > device_path   # 这里关联的存储设备是/dev/sdb,根据实[>
echo 1 > enable
cd /sys/kernel/config/nvmet/ports
mkdir 123
cd 123
ip addr add 192.168.1.159/24 dev eth0    # 192.168.102.25 是本机的IP地址，配置[>
echo 192.168.1.159 > addr_traddr
echo tcp > addr_trtype
echo ipv4 > addr_adrfam
echo 4420 > addr_trsvcid   

cd subsystems/
ln -s ../../../subsystems/nvme-subsystem-name nvme-subsystem-name
```


#### 主机client端
```
modprobe nvmet-tcp
ip addr add 192.168.1.160/24 dev enp3s0
nvme discover -t tcp -a 192.168.1.159 -s 4420
```


## Requirements:  VM with ubuntu and docker/docker-compose installed.  If you don't have that, please look at the bottom of this README file

- Configuration for dhcp:  https://www.thegeekdiary.com/dhcp-configuration-file-etcdhcpdhcpd-conf-explained/
- Got main instructions from: https://github.com/networkboot/docker-dhcpd

###  To get up and running:
1) git clone https://github.com/soumukhe/dhcp-server_docker-compose.git
2) cd dhcp-server_docker-compose
3) cd data ;  vi dhcpd.conf   and fix the subnet, range and default gateway to match your local subnet and range you desire
4) docker-compose up --build -d ( or do docker-compose build   followed by docker-compose up -d)
5) To make changes in dhcp config,  cd data (on local host),  make changes to dhcpd.conf,  docker-compose restart


- To Check if it's running: docker-compose ps

- if needed docker-compose restart, also do this if you make changes to dhcpd.conf
-  where you’ll see DHCP requests, leases granted, and errors:
``` bash
   docker logs -f sm-dhcp
```

-  or you can do:
```bash
docker exec -it sm-dhcp bash
ls -l /var/log | grep dhcp
grep dhcp /var/log/syslog
```

example dhcpd.conf:
---------------------
aciadmin@freeradius:~/DHCP-10.0.140.0/data$ cat dhcpd.conf
#https://www.thegeekdiary.com/dhcp-configuration-file-etcdhcpdhcpd-conf-explained/

## example 1: option definitions common to all supported networks...

```csv
option domain-name  "cloudtiger.local";
option domain-name-servers 8.8.8.8;


default-lease-time 600;
max-lease-time 7200;

subnet 10.0.0.0 netmask 255.255.0.0 {
  range 10.0.140.1 10.0.141.254;
  option routers 10.0.0.1;
}
```
## example 2

#### Here, only `.10` to `.19` are available for general dynamic assignment. The addresses `.100` and `.101` are specifically tied to those MAC addresses and are not part of the dynamic pool

```csv
subnet 10.0.0.0 netmask 255.255.0.0 {
  range 10.0.140.10 10.0.140.19;  # Only 10 addresses for dynamic assignment
  option routers 10.0.0.1;
}

# Static MAC assignments, can use any address within the subnet
host device1 {
  hardware ethernet 08:00:27:AA:BB:CC;
  fixed-address 10.0.140.100;
}

host device2 {
  hardware ethernet 08:00:27:DD:EE:FF;
  fixed-address 10.0.140.101;
}
```

## example 3; Only `device1` and `device2` will get IPs via DHCP, tied to their MAC addresses

```csv
subnet 10.0.0.0 netmask 255.255.0.0 {
  option routers 10.0.0.1;
  # No range defined!
}

host device1 {
  hardware ethernet 08:00:27:AA:BB:CC;
  fixed-address 10.0.140.100;
}

host device2 {
  hardware ethernet 08:00:27:DD:EE:FF;
  fixed-address 10.0.140.101;
}
```

In case you don't have a vm with docker/docker-compose, follow these steps first to install:
--------------------------------------------------------------------------------------------
```bash
1) Download Ubuntu 20.04 --  https://ubuntu.com/download/desktop
2) Install Ubuntu VM minimum versiojn
3) update/upgrade ubuntu and install basic utils: 
    sudo -i
    apt update && apt upgrade -y
    apt auto-remove
    apt install -y curl wget vim openssh-server
 4) Install docker and docker-compose:
    ssh to your Ubuntu VM on backend APIM EPG
    sudo -i
    apt-get update && apt-get upgrade -y
    echo net.ipv4.ip_forward=1 >> /etc/sysctl.conf
    sysctl -p
    sudo sysctl --system
    exit 
    sudo apt install docker.io -y
    sudo systemctl start docker
    sudo systemctl enable docker
    sudo groupadd docker
    sudo usermod -aG docker $USER
    exit # and ssh back in for this to work
    docker --version
    sudo apt install docker-compose -y
```

For newer versions: if docker-compose gives you error:

```bash
sudo apt remove -y docker-compose
sudo apt update
sudo apt install -y docker-compose-plugin
docker compose version instead of docker-compose version
```


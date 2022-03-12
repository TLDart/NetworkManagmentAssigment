# Assignment configs
## Networks
    172.20.1.0 VM1
    172.21.1.0 VM2
    192.168.100.0 VM3
    192.168.200.0 VM4


## Configs for the machines

### Machine 1 (Client 1)
#### Change hostname (in /etc/hostname and then in /etc/hosts)
#### Machiune IP - 192.168.100.1
### #In /etc/network/interfaces
    delete allow hotplug
    auto <interface_name>

### Machine 2 (Client 2)
#### Change hostname (in /etc/hostname and then in /etc/hosts)
#### Machine IP - 192.168.200.1
#### In /etc/network/interfaces
    delete allow hotplug
    auto <interface_name>

### Machine 3 (GW 1)
####Change hostname (in /etc/hostname and then in /etc/hosts)

#### Install route package
    sudo apt install net-tools

#### Set static ip's for the server's interfaces (in /etc/network/interfaces)
#### The loopback network interface
    auto lo
    iface lo inet loopback

    #Config Network Interfaces
    allow-hotplug ens160
    auto ens160
    iface ens160 inet dhcp
            # NAT for all networks (MASQUERADE)
            post-up iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE

    auto ens193
    iface ens193 inet static
            address 10.254.254.1
            netmask 255.255.255.0
            # Route 192.168.200.0/24 network via GW2
            up route add -net 192.168.200.0 netmask 255.255.255.0 gw 10.254.254.2
            # Route 172.21.1.0/24 network via GW2
            up route add -net 172.21.1.0 netmask 255.255.255.0 gw 10.254.254.2

    auto ens161
    iface ens161 inet static
            address 192.168.100.1
            netmask 255.255.255.0

    auto ens256
    iface ens256 inet static
            address 172.20.1.1
            netmask 255.255.255.0


#### Set up the interfaces in which we are listening for dhcp (In/etc/default/isc-default-server)
    INTERFACESv4="ens161 ens256"

    
#### Change config for dns (in /etc/dhcpd/dhcpd.conf)
        option domain-name "cbr.gisi.pt";
        #option domain-name-servers 10.254.254.1, 10.254.254.2;

        default-lease-time 21600;
        max-lease-time 21600;
        min-lease-time 21600;

        ddns-update-style none;
        
        subnet 192.168.100.0 netmask 255.255.255.0 {
            option dhcp-renewal-time 14400;
            option routers 192.168.100.1;
            option subnet-mask 255.255.255.0;
            range 192.168.100.10 192.168.100.100;
        }

        subnet 172.20.1.0 netmask 255.255.255.0 {
            option dhcp-renewal-time 14400;
            option routers 172.20.1.1;
            option subnet-mask 255.255.255.0;
            range 172.20.1.10 172.20.1.100;
        }

        host mail-server{
            option host-name "mail-server.cbr.gisi.pt"; #Static Ip for the mail server
            option routers 172.20.1.1;
            hardware ethernet 00:0c:29:8c:8d:52;
            fixed-address 172.20.1.5;
        }

#### Configuring Bind for DNS
#### In /etc/bind/named.conf.local
    zone "gisi.pt" IN {
            type master;
            file "/etc/bind/gisi.db";
            allow-transfer {10.254.254.2;};

    };

    zone "cbr.gisi.pt" IN{
            type master;
            file "/etc/bind/gisi_cbr.db";
            allow-transfer {10.254.254.2;};
    };

    zone "lx.gisi.pt" IN{
            type master;
            file "/etc/bind/gisi_lx.db";
            allow-transfer {10.254.254.2;};
    };

    zone "1.20.172.in-addr.arpa." IN{
            type master;
            file "/etc/bind/cbr1_rev.db";
            allow-transfer {10.254.254.2;};
    };

    zone "100.168.192.in-addr.arpa." IN{
            type master;
            file "/etc/bind/cbr2_rev.db";
            allow-transfer {10.254.254.2;};
    };

    zone "1.21.172.in-addr.arpa." IN{
            type master;
            file "/etc/bind/lx1_rev.db";
            allow-transfer {10.254.254.2;};
    };

    zone "200.168.192.in-addr.arpa." IN{
            type master;
            file "/etc/bind/lx2_rev.db";
            allow-transfer {10.254.254.2;};
    };

### Machine 4 (GW 2)
##### Change hostname (in /etc/hostname and then in /etc/hosts)

#### Install route package
    sudo apt install net-tools

#### Set static Ip for the server ( in /etc/network/interfaces)

#### The loopback network interface
    auto lo
    iface lo inet loopback

#### Config Network Interfaces
    allow-hotplug ens160
    auto ens160
    iface ens160 inet dhcp
            # NAT for all networks (MASQUERADE)
            post-up iptables -t nat -A POSTROUTING -o ens160 -j MASQUERADE
    auto ens193
    iface ens193 inet static
            address 10.254.254.2
            netmask 255.255.255.0
            # Route 192.168.100.0/24 network via GW1
            up route add -net 192.168.100.0 netmask 255.255.255.0 gw 10.254.254.1
            # Route 172.20.1.0/24 network via GW1
            up route add -net 172.20.1.0 netmask 255.255.255.0 gw 10.254.254.1

    auto ens256
    iface ens256 inet static
            address 192.168.200.1
            netmask 255.255.255.0

    auto ens161
    iface ens161 inet static 
            address 172.21.1.1
            netmask 255.255.255.0

#### Set up the interfaces in which we are listening for dhcp (In/etc/default/isc-default-server)
    INTERFACESv4="ens161 ens256"

#### Change config for dns (in /etc/dhcpd/dhcpd.conf)
#### option definitions common to all supported networks...
    option domain-name "lx.gisi.pt";
    #option domain-name-servers 101.254.254.1, 10.254.254.2;

    default-lease-time 21600;
    max-lease-time 21600;
    min-lease-time 21600;


    ddns-update-style none;

    subnet 192.168.200.0 netmask 255.255.255.0 {
                    option dhcp-renewal-time 14400;
                    option routers 192.168.200.1;
                    option subnet-mask 255.255.255.0;
                    range 192.168.200.10 192.168.200.100;
    }
    subnet 172.21.1.0 netmask 255.255.255.0 {
                option dhcp-renewal-time 14400;
                option routers 172.21.1.1;
                option subnet-mask 255.255.255.0;
                range 172.21.1.150 172.21.1.200;
    }

    host mail-server{
        option host-name "mail-server.lx.gisi.pt"; #Static Ip for the mail server
        option routers 172.21.1.1;
        hardware ethernet 00:0C:29:76:F2:A4;
        fixed-address 172.21.1.5;
    }
#### Configuring Bind for DNS
#### In /etc/bind/named.conf.local
    zone "gisi.pt" IN {
            type slave;
            file "/etc/bind/gisi.db";
            masters {10.254.254.1;};

    };

    zone "cbr.gisi.pt" IN{
            type slave;
            file "/etc/bind/gisi_cbr.db";
            masters {10.254.254.1;};
    };

    zone "lx.gisi.pt" IN{
            type slave;
            file "/etc/bind/gisi_lx.db";
            masters {10.254.254.1;};
    };

    zone "1.20.172.in-addr.arpa." IN{
            type slave;
            file "/etc/bind/cbr1_rev.db";
            masters {10.254.254.1;};
    };

    zone "100.168.192.in-addr.arpa." IN{
            type slave;
            file "/etc/bind/cbr2_rev.db";
            masters {10.254.254.1;};
    };

    zone "1.21.172.in-addr.arpa." IN{
            type slave;
            file "/etc/bind/lx1_rev.db";
            masters {10.254.254.1;};
    };

    zone "200.168.192.in-addr.arpa." IN{
            type slave;
            file "/etc/bind/lx2_rev.db";
            masters {10.254.254.1;};



#### Machine 5 (Server 1)
    #Change hostname (in /etc/hostname and then in /etc/hosts)
    #IP 172.20.1.1
    #In /etc/network/interfaces
    #Delete the line allow hotplug
    auto <interface_name>

#### Machine 6 (Server 2)
    #Change hostname (in /etc/hostname and then in /etc/hosts)
    #IP 172.21.1.1
    #In /etc/network/interfaces
    #Delete the line allow hotplug
    auto <interface_name>



#### SSH Passwordless Auth
    #Creating the key 
    ssh-keygen -t rsa -b 4096 -C "duartedias@student.dei.uc.pt"
    ssh-copy-id <destination_ip>
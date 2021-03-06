# How to install pihole/unbound as a recursive DNS server. (Ubuntu 20.04)

### Install Pi-Hole
`sudo curl -sSL https://install.pi-hole.net​ | bash`

### Set the Web Admin Password
`pihole -a -p <password>`

### Install Unbound DNS
`sudo apt install unbound`

### Create Unbound Configuration File
`sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf`

### Copy example config from https://docs.pi-hole.net/guides/dns/unbound/
```
server:
    # If no logfile is specified, syslog is used
    # logfile: "/var/log/unbound/unbound.log"
    verbosity: 0

    interface: 127.0.0.1
    port: 5335
    do-ip4: yes
    do-udp: yes
    do-tcp: yes

    # May be set to yes if you have IPv6 connectivity
    do-ip6: no

    # You want to leave this to no unless you have *native* IPv6. With 6to4 and
    # Terredo tunnels your web browser should favor IPv4 for the same reasons
    prefer-ip6: no

    # Use this only when you downloaded the list of primary root servers!
    # If you use the default dns-root-data package, unbound will find it automatically
    #root-hints: "/var/lib/unbound/root.hints"

    # Trust glue only if it is within the server's authority
    harden-glue: yes

    # Require DNSSEC data for trust-anchored zones, if such data is absent, the zone becomes BOGUS
    harden-dnssec-stripped: yes

    # Don't use Capitalization randomization as it known to cause DNSSEC issues sometimes
    # see https://discourse.pi-hole.net/t/unbound-stubby-or-dnscrypt-proxy/9378 for further details
    use-caps-for-id: no

    # Reduce EDNS reassembly buffer size.
    # Suggested by the unbound man page to reduce fragmentation reassembly problems
    edns-buffer-size: 1472

    # Perform prefetching of close to expired message cache entries
    # This only applies to domains that have been frequently queried
    prefetch: yes

    # One thread should be sufficient, can be increased on beefy machines. In reality for most users running on small networks or on a single machine, it should be unnecessary to seek performance enhancement by increasing num-threads above 1.
    num-threads: 1

    # Ensure kernel buffer is large enough to not lose messages in traffic spikes
    so-rcvbuf: 1m

    # Ensure privacy of local IP ranges
    private-address: 192.168.0.0/16
    private-address: 169.254.0.0/16
    private-address: 172.16.0.0/12
    private-address: 10.0.0.0/8
    private-address: fd00::/8
    private-address: fe80::/10
```

### Restart Unbound to apply Configuration
`sudo service unbound restart`

### Disable Forwarding DNS in PiHole

### Set Custom DNS in PiHole
`127.0.0.1#5335`


# Install wireguard

## *On the Server side:*
### Make sure system is up to date
`sudo apt update && sudo apt upgrade -y`

### Enable IP Forwarding by editing /etc/sysctl.conf
`sudo nano /etc/sysctl.conf`

### Uncomment the line _net.ipv4.ip_forward=1_ (*following command needs to be tested.*)
`sudo sed -i 's/#net\.ipv4\.ip_forward=1/net.ipv4.ip_forward=1/' /etc/sysctl.conf`

### Apply settings
`sudo sysctl -p`

### Install wireguard
`sudo apt install wireguard`

### Generate keys
`wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey`

### Create config file
`sudo nano /etc/wireguard/wg0.conf`

### Add this to the file
```
[Interface]
PrivateKey = <Your Private Key Goes Here>
Address = 192.168.69.1/24
ListenPort = 51820
PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
```
### _You need to replace the PrivateKey with yours and replace eth0 with your interface name_
### You can check what your interface is by typing `ip a` and figuring out which one you want to use.

### Test that the config is OK by bringing up the wg0 interface. Bring it down afterwards.
```
sudo wg-quick up wg0
sudo wg-quick down wg0
```
### You can enable it on boot
`sudo systemctl enable wg-quick@wg0`

## *On the Client side:*

### Install wireguard and create keys (Linux exemple)
```
sudo apt install wireguard
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

### Copy the public key of the client in the config file of the server. It should now look like this (server-side):
```
[Interface]
 PrivateKey = <This Ubuntu Servers Private Key>
 Address = 192.168.69.1/24
 PostUp = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
 PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
 ListenPort = 51820
 

[Peer]
# Test Debian Client
 PublicKey=<The Public Key of the Debian Client>
 AllowedIPs=192.168.69.2 # This is the given IP of the client
 PersistentKeepalive=25 # You might want to comment or remove this line if you don't need to have a keepalive
```

### Create a config file on the client
`sudo nano /etc/wireguard/wg-client.conf`
```
[Interface]
PrivateKey = <This Debian Client Private Key Goes Here>
 Address=192.168.69.2/24

[Peer]
# Ubuntu Server
 PublicKey=<Public Key From Ubuntu Server>
 Endpoint=<Public IP of Ubuntu Server>:51820
 AllowedIPs = 0.0.0.0/0 # Forward all traffic to server
```

### Recommended: enable ufw firewall
```
sudo ufw allow 22/tcp
sudo ufw allow 51820/udp
sudo ufw allow 53
ufw enable
```

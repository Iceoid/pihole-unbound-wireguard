# How to install pihole/unbound as a recursive DNS server. (Ubuntu 20.04)

### Install Pi-Hole
`sudo curl -sSL https://install.pi-hole.net​ | bash`

### Set the Web Admin Password
`pihole -a -p <password>`

### Install Unbound DNS
`sudo apt install unbound`

### Create Unbound Configuration File
`sudo nano /etc/unbound/unbound.conf.d/pi-hole.conf`

### Copy example config from https://www.youtube.com/redirect?event=video_description&redir_token=QUFFLUhqbGctSGkxTUduUjkzX1lWNko4N3JiQUlidWIwQXxBQ3Jtc0ttekZJX1pSdHR2WElkRGhoVDFTaTBBZ21vN3V1THN0SjJuT1dDeGp5SWNCY0dVRjExM3ZZWTgxUkRDV2NGM0dwLVpHajlDdm42bUJ6NVdRdE9UYWlvRnlaWThsMDc5MEtxQVh3REhzYU9GMThnQ1VuTQ&q=https%3A%2F%2Fdocs.pi-hole.net%2Fguides%2Fdns%2Funbound%2F
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

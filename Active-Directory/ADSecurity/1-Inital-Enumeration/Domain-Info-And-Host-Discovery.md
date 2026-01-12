# Gathering Fundamental Domain Information & Discovering Hosts

### Host Discovery:

As soon as we appraoch and Active Directory enviornment, we should begin by running `nxc smb <ip-range>` or a similar *ICMP scanning* tool to quickly find all possible alive machines in the network. 

I like netexec because it provides additional information about the machines it finds such as their host name, OS information, and domain name. With that said, it likely takes a lot longer than a pure ICMP scanner that we could potentially run with Naabu. Just depends on our specific needs.

We really also want to use an *ARP scan* as well to find devices that may not support ICMP (they will be useful, trust me), such as IoT devices. 


### Basic Domain Information Gathering

We also need to grab some fundamental domain information as we will need it for most attacks we will use in the future. Namely, we need the domain name and the IP address of the domain controller(s). 

We have a multiude of methods to get the domain controller(s) IP address

1. Netexec host discovery output 
    - Since the output provides the hostname we may be able to easily spot the domain controller IP there
2. /etc/resolv.conf  
    - Once we land in an AD enviornment via a VPN, pluggable, or by joining local wifi, we should look at our /etc/resolv.conf file on our 'attacker' machine
        - DHCP is typically used in AD as a way for clients to seamlessly get their AD specific DNS settings
        - The domain controller is typically the DNS server and can also be the DHCP server
        - This means that once we join the network, an entry should be added to /etc/resolv.conf that contains the domain name and the domain controller IP, so long as the domain controller functions as the DNS server
    - If we don't see the domain name in /etc/resolv.conf we can use the new nameserver IP that was added and use that to do a reverse dns lookup 
eg `nslookup 10.80.80.2` where that IP is the new entry in /etc/resolv.conf
        - Better yet, we should do  `nxc smb <ip>` with the same IP to get more info on the DNS server to really help us pintpoint if it is the DC or not
3. Listing all Domain Controller IPs with *dig*
    - `dig +short _ldap._tcp.dc._msdcs.<domain> SRV` - This should provide all DC IPs on the AD network
4. Packet Sniffing 
    - We can simply start sniffing network traffic looking for kerberos or LDAP requests (port 88 adn 389 respectively) these often contain the domain name in all uppercase and tell us the domain controller IP too
    - Not super practical, but its a cool technique 


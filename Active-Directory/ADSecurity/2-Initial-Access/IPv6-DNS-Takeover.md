# IPv6 DNS Takeover

This is one of my favorite attacks for AD. It can enable us to capture credentials, relay, enabled authentication coercion, so much cool stuff, and its VERY reliable. 

### High-Level Overview

IPv6 is enabled by default and preferred over IPv4 on Windows, but not actually set up or configured properly on many networks, making it a close-to-perfect technology to target. 

Many common Windows authentication flows involving SMB, HTTP, RPC, or LDAP will create IPv6 DNS requests. This goes back to the name resolution stuff; those name resolution requests get sent over IPv6 constantly.

Different from the LLMNR stuff, IPv6 DNS responses have first preference over everything, even over IPv4 DNS responses. **So if we can respond really fast to the IPv6 DNS query or if IPv6 is not configured in the network, we will win the race.**

An attacker that's on the network can set up a server to listen for these IPv6 DNS requests and respond to them. Since IPv6 is not configured on the network, anyone can respond. 

The machine that initiated the authentication now thinks the IP in the DNS response we provided is the IP of a legitimate Active Directory service, and will proceed with the authentication flow to that specified IP.

### Conditions Required for this to Work

- IPv6 is enabled on victim machines (default on Windows machines)
- Client must accept attacker-controlled IPv6 information (this is the core of the vulnerability)
  - No real v6 router advertisements exist
  - Real v6 RAs exist, but ours has better metrics or explicitly advertises DNS, while the real RA does not
  - RA guard is not enabled on switches
  - DHCPv6 is not enforced
- Client sends traffic to attacker-controlled DNS
  - The attacker becomes the IPv6 DNS server
  - Clients send AAAA lookups
  - Attacker replies with any address
  - Client believes attacker is the DC
- The service used by the client supports NTLM
  - This is where the authentication happens
  - Common ones are SMB, LDAP, HTTP, RPC
- NTLM is allowed/ network only allows Kerberos

### Attack Flow

Do you see any similarities between the initial attack vectors yet?

1. Victim needs to authenticate for some reason, like logging in, accessing a file share, refreshing GPO, using RPC, etc, all use name resolution
2. The victim initiates the name resolution to find the IP of the server they want to authenticate to
3. Windows will ask for the IPv6 AAAA DNS records FIRST because IPv6 is preferred
4. Attacker replies, telling the victim that their IP is the IP of the server they want to authenticate to
5. Victim machine accepts the response; no authentication exists here, IPv6 is preferred, and the response looks normal
6. Victim proceeds to authenticate to the attacker

This attack is just another way to put ourselves in a position to receive authentication from AD users


### Performing the Attack - Commands and Tools

***THIS ATTACK MUST ONLY BE RAN FOR VERY SHORT PERIODS OF TIME. IT CAN BRING DOWN NETWORKS.***

We have a lot of options depending on what exploit we want to use, often we will want to perform relays, which we use Impacket's NTLMRelayX to perform. 

Example: `impacket-ntlmrelayx -6 -i -t ldaps://10.80.80.2 -wh fakewpad.<domain> -l lootme` - dumps a ton of info about the domain into a directory called 'lootme' and gives us an interactive shell

This tool does SMB and LDAP relay for sure, likely other protocols too. 

To perform the IPv6 DNS stuff, we will use *MITM6*. There's an interesting tool called *Pretender* out there too, but I haven't got a chance to check it out much. 

Again, we must be on the same network for this attack since its essentially a man in the middle. 

To run MITM6 just do `sudo mitm6 -d <domain>`

I think we may even be able to use *bettercap* to do this, too, but again, not positive. 


### Defenses and Mitigations

1. One method is to disable IPv6 internally, but this will almost certainly break some stuff or at least cause weird things to happen at least at first. It's possible to successfully disable IPv6, but it will take some time and may not be possible depending on your network.
2. Firewall rules should be put in place to prevent these attacks. Best bet is to restrict IPv6 authentication traffic to domain controllers only and block IPv6 DNS except traffic involving real servers
3. We should also be disabling WPAD if it's not in use, as this creates DNS queries that often aren't needed
4. Again, require signing everywhere to prevent relays
5. Move all admin accounts to the protected users group and/or mark the accounts as sensitive, which prevents impersonation attacks in some cases
6. RA Guard - IPv6 takeover begins with malicious router advertisements; RA Guard will block unauthorized RAs at layer 2. Very important: RA Guard is configured properly (handles fragmentation packets and extension headers)
7. Disable NTLM where possible, require NTLM signing, deny NTLM to DCs
8. Monitor RA traffic and look for rogue RAs, unexpected IPv6 responses, NTLM authentication to non-DC hosts, and authentication over IPv6 from workstations



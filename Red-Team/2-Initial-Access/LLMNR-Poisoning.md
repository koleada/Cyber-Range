# LLMNR & NBT-NS Poisoning

LLMNR poisoning is one of the most well-known and reliable attacks in Active Directory penetration testing.

It's most effective when run first thing in the morning or right after lunch, when everyone is logging in and getting back into their work environment.

### Attack Overview

This attack happens when DNS fails in a network. When that happens, Windows will fall back to older resolution protocols. 

General Resolution Protocol Order:
1. DNS (via configured DNS server)
2. LLMNR
3. mDNS (not as important in an AD environment)
4. NBT-NS
5. Hosts file or cache

LLMNR and NBT-NS quieres get sent to an arbitrary number of devices on the local network. 
Any device can respond to these queries and specify the IP address of the host being requested. 
Windows does not do any verification on the response at all. 

This is all default Windows behavior; both protocols are enabled, and Windows automatic trust is how the protocols should work according to Microsoft.

Windows will automatically initiate authentication flows involving SMB, HTTP, or LDAP when the user performs certain actions, like trying to access a file share. 
If the share, for example, that a user wants to access contains a typo and fails to resolve, Windows will fall back to LLMNR. 
An attacker on the same subnet can respond to the LLMNR request saying the share can be found on their machine. 
The victim machine will trust that response and then proceed to automatically authenticate via NTLM to the attacker's machine. 
This results in the attacker being able to capture the user's username and password hash to be cracked or passed around.

### The Protocols

1. Link-Local Multicast Name Resolution (LLMNR)
   - Purpose: Perform name resolution in a local network when DNS is unavailable or broken
   - UDP protocol, port 5355, link-local scope
   - General LLMNR Flow: 
     1. DNS fails to resolve a hostname
     2. Windows sends a multicast LLMNR query trying to resolve the hostname
     3. Since it's a multicast packet, it is sent to all devices on the local subnet so long as they are listening to this address: 224.0.0.252
     4. Any device can respond to this LLMNR query; again, there is no validation whatsoever. The first response is the winner
     5. Windows then trusts the response and proceeds 
2. NetBIOS Name Service (NBT-NS)
  - Purpose: Again, NBT-NS is designed to perform name resolution when DNS fails
  - UDP protocol, port 137, broadcast scope
  - General NBT-NS Flow:
    1. Hostname fails to be resolved via DNS and LLMNR (in some cases, mDNS too)
    2. Windows sends a NetBIOS name query looking for the host
    3. The packet gets sent to every device on the subnet since it's a broadcast
    4. Again, any host that receives the packet can respond to it
    5. Windows will accept the response without validation and proceed with the flow
  - NBT-NS is 'worse' than LLMNR, its broadcast, so it reaches more devices. and it's older
3. NT LAN Manager (NTLM)
  - Purpose: NTLM is a suite of Microsoft security protocols that are used for authentication. It provides a challenge-response authentication framework, which removes the need to send clear-text passwords.
  - Challenge-Response Flow
    1. Victim attempts to access a resource, for example \\TEST
    2. The device automatically searches for the device hosting the share
    3. The hosting device responds, or its IP is found with DNS
    4. The client initiates NTLM authentication
       - First, the client sends the username
       - Server sends a challenge nonce in return
       - Client calculates the hash of the password and the challenge and sends it to the server

### Attack Flow, Commands, and Tooling

**Note:** We must be on the same subnet for this attack to work in almost all cases. The LLMNR/NBT-NS queries are link-local/broadcast-based. Routers drop broadcast messages by design, so they will not be routed between subnets.

This attack is fairly easy to perform with a tool called *Responder* by lgandx. 

- `sudo responder -I <interface> -dP` - start responder to enable the capturing of credentials
- The *-w* flag is also interesting; this enables WPAD. WPAD allows Windows devices to automatically discovery proxy settings by looking for a specific host like wpad.<domain> and fetching a configuration file from that server.
  - When fetching that WPAD file, Windows will often try to authenticate to the web server using NTLM if the device is configured to use integrated authentication
  -   a target machine, we can see if WPAD is enabled by going to Control Panel> Internet Options> Connections> LAN settings > "Automatically Detect Proxy Settings"
    - This setting is enabled by default, I believe
  - This will trigger when the victim accidentally browses to a website that does not resolve, causing the machine to make calls to look for the WPAD configuration file
  - Responder can respond with a pop-up on the victim's browser that asks them to log in, and when they do so, it will send the credentials to us
  - Can also do `sudo responder -I eth0 -wdF -b` which forces 'basic authetnication'
    - It's similar to normal WPAD, but will cause the default browser pop-up to appear and will provide us with the cleartext password instead of a hash
    - On chrome both are the same, on edge this basic authentication pop-up is a bit weird
- ** --lm --disable-ess** - We should always be adding this too, it will try to force NTLMv1 and use NTLMv2 without ESS, making the hash much easier to crack (not possible in practice on new Windows versions)


### LLMNR Poisoning Defenses and Mitigations

The first step to eliminating this attack is to turn off LLMNR in our AD environment using group policy
  - Go to "policies/administrative templates/network" then select "turn off multicast name resolution" and enable that
  - Push the policy to the entire domain

We should also set up network access controls so that if someone tries to plug in a remote access device, they cannot just land on our network. Something like MAC address filtering or some degree of network authentication is a great move. Additionally, we could force any untrusted devices to land on an isolated subnet away from important infrastructure.

Having a very strong password policy (enforced to some degree with group policy) is also necessary to make it as hard as possible to crack hashes when they are captured

Finally, we should be carefully monitoring and logging NTLM authentication to untrusted devices, for exmaple those that are not the domain controller, known file servers or webservers, etc. 


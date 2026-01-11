# Cyber Range 

This cyber range is built to resemble an enterprise network while also saving me, as a broke college student, from having to spend a ton of money on real hardware, software licenses, paid labs/ education, and so forth. It is complete with multiple subnets, each containing machines with different purposes, a central firewall that is used as a router, a virtual private network to access a certain subnet, and lots of software tooling and configuration across the various machines. My range has a subnet for the attacker machine, an Active Directory network, a subnet for miscellaneous vulnerable machines, and a subnet for monitoring the Active Directory environment. 

### Project Overview 

My goal with this was to build a full-fledged mini network, both to demonstrate my knowledge to others, help them design their own home lab, and also to learn more about administering and red teaming on 'real' networks. I've always been very interested in active directory so one of my main focuses with this lab was to really dive in to it both from an administration/ configuration standpoint and regarding various cyber attacks and defenses specific to AD. Additionally, I have a goal of becoming a penetration tester and obtaining certifications like the OSCP (once I can afford it), so I figured having a true cyber range set up would help me later in my career as well. Lastly, I wanted to gain experience in going through the full network setup process, starting with the design phase.

### Skills and Experience Gained 

1. **Active Directory initial configuration:**
    - Setting up the domain controller and downloading the necessary tooling
    -  Configuring the DC as a DNS server
    -  Configuring the DC to be a DHCP server
    -  Creating the initial forest  
2. **Active Directory user management:**
   - Adding users
   - Controlling privileges and access
   - Group policy objects, and modifying group policy
   - How to use organizational units and groups to apply policies to multiple objects at once
3. **Virtualization:**
    - Standing up and configuring new VMs
    - When VMs should and should not be used
    - How each different type of VM network adapter works and what the purpose of each is
4. **Networking:**
    - Configuring a router(I know it's technically a firewall) with multiple subnets, understanding subnet masks and IP addressing
    - Understanding DHCP for IP addressing assignment, knowing when and where it should be used, and what software is acting as the DHCP server in which place
    - Understanding Network Address Translation, including when it is needed, how to configure it, and what its purpose is
    - Understanding DNS, particularly concerning Active Directory, knowing why the domain controller must be a DNS server despite needing to use an alternate machine as a DNS server to reach the internet
    - Understanding and using many network protocols like Lightweight Directory Access Protocol, Server Message Block, Link-Local Multicast Name Resolution, NetBIOS Name Service, Address Resolution Protocol, Kerberos, Remote Desktop Protocol, Secure Shell, Remote Procedue Call, NT LAN Manager, Domain Name System, Multicast Domain Name System, Dynamic Host Configuration Protocol (to be clear, by understanding, I mean truly knowing how these things work under the hood and what exactly they do within a network)
5. **Using and Configuring Virtual Private Networks:**
    - Understanding what the purpose of a VPN is and why it is useful
    - Setting up a basic VPN using a VPN client and server configuration file
    - Limitations of using a VPN connection for remote access (particularly from the perspective of an attacker, tooling like Responder is not usable via a VPN connection in most cases)
6. **Active Directory Initial Attack Vectors - How they work, How to perform them, How to defend from them**
    - LLMNR Poisoning - Responder, understanding when LLMNR/NBT-NS/mDNS are used, understanding what causes Responder to actually get a hit, cracking hashes, LLMNR, NBT-NS, and NTLM are ALL enabled by default on Windows machines, LLMNR poisoning coercion techniques via different vulnerabilities (require credentials)
    - SMB Relay - What SMB signing is and why it is so important, how to find machines without SMB signing enabled, ntlmrelayx with responder, ensure signing is enabled on all devices, the importance of following least privilege to mitigate damage if this attack is pulled off
    - IPv6 DNS Takeover - This is my favorite active directory exploit I've learned about so far. IPv6 is enabled but not used on most machines in most networks, making it a close to perfect technology to target, we can listen for IPv6 packets coming through and pretend to be the DNS server since msot networks do not have an IPv6 DNS server. 








    

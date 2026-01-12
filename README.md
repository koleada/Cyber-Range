# Cyber Range 

This cyber range is built to resemble an enterprise network while also saving me, as a broke college student, from having to spend a ton of money on real hardware, software licenses, paid labs/ education, and so forth. It is complete with multiple subnets, each containing machines with different purposes, a central firewall that is used as a router, a virtual private network to access a certain subnet, and lots of software tooling and configuration across the various machines. My range has a subnet for the attacker machine, an Active Directory network, a subnet for miscellaneous vulnerable machines, and a subnet for monitoring the Active Directory environment. 

### Project Overview 

My goal with this was to build a full-fledged mini network, both to demonstrate my knowledge to others, help them design their own home lab, and also to learn more about administering and red teaming on 'real' networks. I've always been very interested in active directory so one of my main focuses with this lab was to really dive in to it both from an administration/ configuration standpoint and regarding various cyber attacks and defenses specific to AD. Additionally, I have a goal of becoming a penetration tester and obtaining certifications like the OSCP (once I can afford it), so I figured having a true cyber range set up would help me later in my career as well. Lastly, I wanted to gain experience in going through the full network setup process, starting with the design phase.

### Overview of Experience Gained and Things Learned

1. **Active Directory Initial Configuration:**
    - Setting up a Windows Server virtual machine to act as the domain controller and including the necessary tooling
    - Promoted the server to a domain controller and created the initial Active Directory forest and domain
    -  Configured the domain controller to be the DNS server for the domain
       - Understanding AD DNS zones and the essential nature of DNS SRV records for authentication and service usage in an AD network
    -  Configuring the Domain Controller to be the DHCP server for domain-joined computers
        - Defining DHCP IP ranges
        - Assigning the default gateway, DNS server, and other necessary domain information via DHCP
    -  Adding machines to the Active Directory network 
    -  Configuring Active Directory Certificate Services
        - Installing and configuring a certificate authority
        - Understanding the security model of certificates and their use in authentication flows
    -  Configured secure communication for common AD protocols
        - Enabling LDAPS to ensure LDAP queries are encrypted
        - Understanding and configuring SMB encryption and signing
    - Learned about how authentication and other AD services rely on time synchronization, DNS resolution, and certificates 
      
2. **Active Directory User Management:**
   - Created and managed domain users, other domain accounts, and security groups
   - Assigned privileges and resource permissions based on group membership
   - Created and modified Group Policy Objects to enforce security controls and general configurations
       - Applied policy to both specific OUs and the entire domain
       - Understanding GPO inheritance, enforcement, and specificity order if conflicting GPOs exist 
   - Used Organizational Units to group users, computers, and service accounts
   - Validated and troubleshot group policies using GPResult and RSoP
   - Learned about common causes of group policy application issues
       - OU placement issues
       - Security filtering
       - DNS or domain authentication issues
  
3. **Active Directory Administration & Operations:**
    - Designing and maintaining an organizational unit layout that follows:
        - Least privilege
        - Separation of admin-related roles
        - Easy policy application
    - Implemented role-based access control in the Active Directory environment by creating different roles (eg, IT, sales, etc) and using AD delegation features
    - Managed machine and service accounts
        - Understanding when computer accounts authenticate to the domain
        - Understanding the risks associated with service accounts and how/why they are used
    - Familiarized with common AD issues, common causes, and common solutions
    - Configuring domain-wide and organizational unit group policy, such as password policy and various security-related configurations
     
3. **Virtualization:**
    - Created and managed multiple connected virtual machines using a type 2 hypervisor (namely VirtualBox)
    - Configured machines for different roles like domain controller, attacker machines, workstations, networking systems, and so forth
    - Learned when virtualization is best implemented and when it should not be used
    - Understood how all the different types of VM network adapter works and what the purpose of each is (NAT, Host-only, bridged, internal, generic, NAT network)
      
4. **Networking:**
    - Configuring a router(I know it's technically a firewall) with multiple subnets, understanding subnet masks and IP addressing
    - Understanding DHCP for IP addressing assignment, knowing when and where it should be used, and what software is acting as the DHCP server in which place, and configuring DHCP
    - Understanding Network Address Translation, including when it is needed, how to configure it, and what its purpose is
    - Understanding DNS, particularly concerning Active Directory, knowing why the domain controller must be a DNS server despite needing to use an alternate machine as a DNS server to reach the internet
    - Understanding and using many network protocols, including:
          - Lightweight Directory Access Protocol
          - Server Message Block
          - Link-Local Multicast Name Resolution
          - NetBIOS Name Service
          - Address Resolution Protocol
          - Kerberos
          - Remote Desktop Protocol
          - Secure Shell
          - Remote Procedure Call,
          - NT LAN Manager
          - Domain Name System
          - Multicast Domain Name System
          - Dynamic Host Configuration Protocol
          - (to be clear, by understanding, I mean truly knowing how these things work under the hood and what exactly they do within a network)
      
5. **Using & Configuring Virtual Private Networks:**
    - Understanding what the purpose of a VPN is and why it is useful
    - Setting up a basic VPN using a VPN client and server configuration file
    - Limitations of using a VPN connection for remote access (particularly from the perspective of an attacker, man-in-the-middle attacks are not viable via a VPN connection in most cases)
  

6. **Active Directory Security**
    - 



# THIS IS ALL GETTING MOVED TO THE AD SECURITY DIR IN THE REPO
      
**Active Directory Initial Attack Vectors**
    - LLMNR Poisoning
         - Responder
         - Understanding when LLMNR/NBT-NS/mDNS are used
         - Understanding what causes Responder to actually get a hit
         - Cracking hashes
         - LLMNR, NBT-NS, and NTLM are ALL enabled by default on Windows machines
         - LLMNR poisoning coercion techniques via different vulnerabilities (require credentials)
           
    - SMB Relay
       - What SMB signing is and why it is so important
       - How to find machines without SMB signing enabled
       - ntlmrelayx with responder
       - Ensure signing is enabled on all devices
       - The importance of following least privilege to mitigate damage if this attack is pulled off
         
    - IPv6 DNS Takeover (this is mysecond favorite active directory exploit)
        - IPv6 DNS requests are often generated during normal authentication flows involving SMB, LDAP, HTTP, or RPC
        - IPv6 is preferred by Windows, but usually not configured in a network
        - An attacker on the network can respond to the IPv6 DNS requests and specify their own IP, resulting in the client machine authenticating to the specified IP
        - Must require signing on SMB and LDAP
        - Must put very specific firewall rules in place to block IPv6 requests on certain protocols
          
    - Passback Attacks
        - How to identify IoT devices on a network via host scanning and fingerprinting
        - Common ways to get access to IoT devices (default credentials, known code execution vulnerabilities, etc)
        - Understanding that many IoT devices do outbound authentication to a domain controller via LDAP or to access a file share, email server, or cloud infrastructure
        - Understanding that changing the specified IP address to the domain controller, file share server, etc can allow us to capture the printer credentials by hosting our own server
          
 **Active Directory Initial Enumeration:**
   - Host Discovery
       - ICMP scans using nmap
       - ARP scans to find IoT devices
       - Netexec host discovery features
         
   - Finding Domain Controller IPs
       - Could simply be found with netexec
       - If DHCP is enabled on the network, we will see the domain name and the DC IP in /etc/resolv.conf
       - Reverse DNS lookups to get the DC hostname
       - Using the dig utility to get all DC IPs, provided we know the domain name
       - Traffic sniffing to potentially determine the DC IP
         
   - LDAP Enumeration
        - Using ldapdomaindump to check for anonymous bind and hopefully gain some information, and potentially enumerate certificate templates too
        - Using nmap's built-in LDAP scripts and the ldapsearch tool
          
   - SMB Enumeration
       - Checking if SYSVOL share is publicly accessible (uncommon but could lead to a quick win) GPP XML files often contain the cPassword field and are easy to decrypt
       - Using nmap or smbmap to enumerate shares on specific machines or IP ranges
       - Using netexec or similar tooling to find anonymous and guest accessible shares
         
   -  User enumeration
         - Looking for anonymous RPC bind to enumerate usernames using the netexec tool is best because it checks for domain enumeration RPCs along withother methods
         - RID brute forcing
         - Kerberos user enumeration (wordlist needed, but can be very useful if there is a standardized username scheme or with custom wordlists, potentially made with publicly accessible emails, etc, lots of ideas for custom wordlists)
           
 *Active Directory Post Compromise:*
   - Post Compromise Enumeration
       - Using ldapdomaindump to grab a ton of information about the AD environment
       - Using Bloodhound to dump Active Directory data, visualize the data in a graph, and query the data we gather
     
   - Authentication Coercion
       - Finding RPC endpoints with rpcdump
       - Standing up a server to capture coerced authentication
       - Using tooling to programmatically attempt to coerce authentication via known RPC pathways (PetitPotam, PrinterBug, etc)
        
    - Lateral Movement
         - Pass the password or pass the hash using netexec or similar tooling
               - netexec makes it super easy to test a credential on a wide range of machines very quickly
               - We want to try this on many different protocols like LDAP, SMB, RPC, RDP, HTTP, WinRM, depending on what were passing
         - Pass the ticket (a lot harder to dump the LSASS than to get NTLM hashes, but stilla  potential vector, albeit maybe later on in the engagement), mostly on the same protocols, but a couple additional ones too
           
    - Gaining More Passwords
        - secretsdump.py - attempts to dump a ton of information off of machines (goal is to be able to dump NTDS.dit, which requires domain admin privileges by default)
        - Mimikatz - super important, need SYSTEM or local admin, but it dumps the LSASS, which allows us to perform pass the ticket attacks, which is a huge step to domain admin
        - cPassword credential harvesting - this was patched a while ago, but unless the files were deleted post-patch, they will remain on the machine, always worth checking this, can be done real fast with Metasploit or similar tooling
        - LNK File Watering Hole
            - Create a malicious file that, when clicked, will send authentication to an IP of our choosing (just requests a share on our attacker's IP; this can be made with a couple of lines of PowerShell)
            - Place a malicious file into a file in a share that gets a lot of traffic, run an SMB server (like Responder) that will capture the authentication attempt when the file is clicked
        - Kerberoasting
            - Attempting to get all SPNs on the domain and then request a TGS for each
            - Cracking the TGS offline in hopes of getting a cleartext password to be passed
            - How Kerberos works under the hood
      
    - Windows Privilege Escalation (super important to gain system on a machine so we can dump LSASS, SAM, etc)
       -  winPEAS to get an initial outline of what techniques are most likely to work on a specific machine
       - Certificate Templates (This is my favorite AD exploit ever)
           - Allows us to create a new certificate in which we can specify an arbitrary subject alternative name(SAN) (likely of an admin or service account), which we can then use to change the password of another domain account with elevated privileges
           - Dump all certificates on a machine with certutil
           - Review the dumped certificates (or better yet, we should make a nice script to do it), looking for a couple of very specific things depending on the exact certificate template privilege escalation we want to use (can use certip-ad or certify to look for misconfigurations, but I have not tested these much yet)
           - Creating a new certificate using the GUI/CLI when/if we are able to find a suitable template
           - Using Rubeus to get a TGT using the service/admin account we specified in the certificate, the certificate file we downloaded, the certificate file password, and a couple of other basic pieces of information
           - Again using Rubeus, but this time to change the password of a domain admin using the TGT we just got
           - In some cases, we can relay NTLM authentication to the AD CS to create a certificate for an arbitrary user
           - If we encounter the 'ENROLL' permission set to 'Everyone' or 'Anonymous Logon', we can create a certificate for any account we know the username of - can check this by browsing to 'http://<target-ADCS>/certsrv' and seeing if the page loads
      
       - DLL Side-Loading
           - Windows uses a specific order of directories when searching for DLLs referenced in an executable
           - If Windows finds a DLL earlier in the search order, that is the one that is used, even if it's not the one the developer intended
           - Find services or executables running as SYSTEM, check if the directory it's in or other directories in the Windows search order are writeable, if so, put a malicious DLL in the directory that is searched earliest, trigger execution, DLL gets executed as SYSTEM
         
       - Scheduled Tasks (Windows version of Cron priv esc)
           - Dump and enumerate all scheduled tasks on a machine, focusing on those that run as SYSTEM/admin
           - Check permissions of the binary or script
           - Replace or modify the code to include malicious privilege escalation stuff
         
       - Stored Credentials, as we elevate privledges we have more ability to dump various stored credentials, doing this on all machines in a network, which can be a great way to further elevate

    - Token Impersonation   ****TODO****
    - Golden Ticket     ****TODO******





    

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
    - Gained understanding of how all the different types of VM network adapter works and what the purpose of each is (NAT, Host-only, bridged, internal, generic, NAT network)
        - Including how virtual networking modes affect network isolation, internet accessibility, and intra-VM communication
    - Used virtualization to affordably and effectively simulate a subnetted enterprise network environment  
      
4. **Networking:**
    - Configured a router/firewall machine with multiple interfaces to create segmented subnets
    - Designed and created multiple subnets, specifically, active directory subnet, an attacker subnet, a detection/monitoring subnet, a cyber range subnet, and a WAN. 
    - Developed a strong understanding of IP addressing and subnetting through configuring my network
    - Gained understanding of and configured DHCP services
        - When DHCP should be centralized
        - When static IP addressing should and should not be used
        - DHCP relations to DNS and Active Directory
    - Learned about NAT, when it is needed, how to configure it, and how it enables private networks to access external resources
    - Learned about Port Forwarding, when  it is needed, how to configure it, and how it enables servers to be accessed by others via the internet  
    - Understanding DNS, particularly concerning Active Directory, knowing why the domain controller must be a DNS server despite needing to use an alternate machine as a DNS server to reach the internet
    - Understanding and using many network protocols, including:
          - Lightweight Directory Access Protocol and its secure version
          - Server Message Block and signing/encryption
          - Link-Local Multicast Name Resolution
          - NetBIOS Name Service
          - Address Resolution Protocol and its importance in LANs
          - Kerberos authentication
          - Remote Desktop Protocol
          - Secure Shell
          - Remote Procedure Call and its relation to Windows services
          - NT LAN Manager
          - Domain Name System
          - Multicast Domain Name System
          - Dynamic Host Configuration Protocol
          - Multicast Domain Name System
          - (to be clear, by understanding, I mean truly knowing how these things work under the hood and what exactly they do within a network)
   - Learned about the difference between north-south and east-west traffic flows and why restricting lateral movement is so important
   - Implemented firewall rules following least-privilege principles to control traffic between subnets, to the internet, and to prevent known exploits
      
5. **Using & Configuring Virtual Private Networks:**
    - Understanding what the purpose of a VPN is and why it is useful
        - Secure remote access
        - Network segmentation and isolation
        - Extending internal networks over infrastructure we do not control 
    - Configured a VPN server and client using OpenVPN
        - Creating client configuration files
        - Basic authentication and encryption 
    - Used a VPN to securly access internal subnets from a remote machine
    - Gained understanding of limitations of VPN connections from a security perspective
        - MITM attacks are not feasible over most VPN connections
        - How VPN access changes the attack surface compared to dropping in directly to the internal network
    - Saw how VPN traffic interacts with routing, firewall rules, and subnet access controls 
  

6. **Active Directory Security:**
    - Understood, performed, and defended modern initial attack vectors
        - LLMNR/NBT-NS poisoning
        - SMB Relay and the importance of enforcing signing everywhere
        - IPv6 DNS takeover attacks from misconfigurations
        - Passback attacks via IoT and network device authentication
    - Learned about and performed Active Directory initial enumeration & reconnaissance
        - Host discovery (ICMP, ARP, and scanning tools)
        - Domain controller and domain information discovery
            - Domain name, domain controller IP address, services
        - SMB and LDAP enumeration via public shares and anonymous bind, respectively
        - User enumeration via RPC, RID brute force, and Kerberos enumeration
    - Learned and performed Active Directory Security Post Compromise
        - Used Bloodhound and ldapdomaindump for environment mapping and detailed information gathering
        - Learned and performed authentication coercion techniques like PrinterBug and PetitPotam
        - Performed lateral movement via pass-the-hash, pass-the-ticket, and pass-the-pass
        - Gained additional credentials with Mimikatz, secretsdump.py, and Kerberoasting
        - Learned about and used Windows privilege escalation techniques
            - Certificate templates
            - DLL side-loading
            - Scheduled tasks
    - Saw firsthand the importance of Active Directory security techniques
        - Least privilege, SMB signing, strong group policy
        - Securing IoT and network devices to prevent passback attacks
        - Role of certificates and AD CS in secure authentication
        - Importance of staying very up to date with Windows patches and security information
   
8. **Monitoring, Detection, & Logging:**

      





    

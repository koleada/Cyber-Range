# Domain Controller Purpose & Services

The domain controller(s) is the core of an Active Directory environment. 

The DC holds the following roles in Active Directory:

- Authentication of users via Kerberos and or NTLM
- Directory lookups for users, groups, computers, service accounts, and other objects
- Enforcement and configuration of group policy
- Replication of other DCs within the domain
- Certificate services if the AD CS is enabled
- Central point for user management, creating, adding, and modifying users and objects
- Various network services, like DNS, DHCP, LDAP, and usually many others too

If the domain controller is comrpomised it will typically allow the attacker to have close to, if not full control over the domain, making them the highest value target. 

*** 

### DNS Services

DNS is a necessary part of any Active Directory network. 

Domain controllers usually function as the directory DNS server to host DNS for:

- Resolving internal domain names
- Supporting Kerberos service principal names (SPNs)
- Enabling LDAP and other foundational AD services

Security Notes:

- Misconfigured DNS is a potentially MASSIVE problem in an Active Directory network, both regarding IPv4 and v6. It can opne your network up to spoofing and many credential intercepation attacks.
- Internal DNS should always be kept internal; we do not want it exposed to untrusted networks

***

### DHCP 

I chose to use DHCP services on the domain controller. 

I just wanted to see how to configure it on the domain controller, as I had already done it on the firewall. 

DHCP just allows IPs to be automatically assigned to other devices on the network. 

***

### FSMO Roles

Flexible Single Master Operation (FSMO) roles are critical for a properly functioning Active Directory network. 

The 5 roles are:
1. Schema Master - controls updates to direcotry schema and is unique to a forest
2. Domain Naming Master - manages the addition and removal of domains in a forest
3. RID Master - handles requests for Relative IDs (RIDs) from domain controllers within a domain, ensuring unique Security IDs (SIDs) for object
4. PDC Emulator - primary domain controller for backward compatibility and manages time synchronization and password changes within a domain
5. Infrastructure Master - updates references to objects in other domains

For my lab, all of these FSMO roles are handled on a single domain controller, though in larger enterprises, they are often split up between controllers. Still, it is very important to be aware of these core roles of domain controllers. 

***

### Time Synchronization 

When first jumping into this, I was surprised to learn how detrimental DNS and time sync are to AD. These are very common causes for a variety of issues that AD users or admins can experience. 

Domain controllers provide NTP (Network Time Protocol) to all domain members. Accurate time is essential for:
- Kerberos authentication
- Certificate validation
- Event correlation
- Probably a lot of other things I'm not aware of, too

An incorrect time can lead to Kerberos failures and replay attacks. 

***

### Replication 

DCs are responsible for replicating directory informtion for a domain to keep consistancy across that domain. 

- Replication ensures all domain controllers have up-to-date user, group, GPO, and other directory information
- In a lab enviornment a single domain controller is fine, but understanding the importance of replication is super important
- Replication traffic should be monitored for oddities or unauthorized modification



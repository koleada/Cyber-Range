### Group Policy (GPO) - Design, Implementation, & Security Controls

Group policy is one of the most powerful components of Active Directory. We touched on it briefly regarding the identity model, but here we will go a lot further.

In my cyber range AD environment, Group Policy is not just used as centalized configuration and management console, it's also used as a primary security control plane that I became familiar with using to modify and secure authentication behavior, privilege boundaries, network security, and attack surface. 

In this doc, I will detail how I learned and performed designing, structuring, and applying GPO within my lab enviornemnt specifically by explaining the following:

1. Practice GPO design principles
2. Policy processing behavior and scope
3. Identity and authentication controls
4. Workstation and server configuration enforcement
5. Foundational security hardening using Group Policy

***

## Foundational Group Policy Concepts 

### Purpose of Group Policy in Corporate Active Directory Networks

From what I've learned, Group Policy is generally used for the following functions:

- Enforce standardized system configuration requirrements accross an entire network of machines at once
- Define and enforce authentication and authorization requirements and behaviors within the AD environment
- Control code execution and system capabilities for specific Organizational Units, domains, etc
- Reduce attack surface within the network and apply baseline hardening strategies across all domain devices
- Support compliance and auditing via logging and monitoring options, and by having a centralized console to view all policies applied within a domain

Within the AD network in my home lab, I practiced and learned about using group policy to mitigate or eliminate the possibility of various exploits and to apply least privilege principles to different accounts across the network. I also intentionally configured bad group policies for certain things to allow certain exploits to be performed for monitoring and research purposes. Additionally, I learned about foundational policies that I believe all AD environments should strive to have and initial hardening strategies for the environment. 

*** 

### Group Policy Hierarchy 

Before implementing Group Policy within a network, one must understand the processing order of Group Policy. By processing order, I'm referring to the hierarchy of how Group Policy handles conflicting policies that are applied to a given subject (OU, domain, etc). This is essential for troubleshooting Group Policy, like when we are not sure why a specific policy is or is not being applied, or determining where inherited behavior is coming from, etc. 

Group Policy Objects are applied based on specificity, where the most specific rule takes precedence over the less specific ones. 

The order (from least specific to most) is as follows:

- Local Computer: There is a local (non-AD) GPO on all Windows machines. This is completely outside of Active Directory, can only be managed on the one machine the policy applies to, and is stored locally. This is the least specific GPO.  
- Site: A site is a group of subnets typically used to represent a physical office location, so different locations get different sites. Sites have higher precedence than local, and these are managed through AD.
- Domain: Domain-wide policy gets applied to all users and computers in said domain. Some policies can only be applied at the domain level; these policies are located in *Default Domain Policy*, but admins will still often apply policies outside of these domain-wide. More specific than site-wide and local computer policy.
- Organizational Unit: Organizational Units exist specifically for applying policy and delegation. They are essentially used to group accounts (Users, Machines, Services, etc) and apply policy to all accounts in that OU at once. There is hierarchy within OUs because OUs can reside within other OUs. The rule is that the 'deepest' OU gets precedent. So if we have a Users OU(parent) and within it a Sales OU(child), policies applied to the Sales OU would take precedent over those applied to the Users OU. OUs are the most specific GPO link targets.

The acronym to help remember this order is *LSDOU*. 

When a domain-joined computer or user logs in, Windows first applies the local group policy, then site wide group policy, then the domain group policy, then the OU group policy, beginning at the parent OU and moving through all child OUs. The deepest OU will have the most precedent by default. As Windows proceeds downward, if confliciting policy appears, the more specific policy will override the less specific one. 

In my lab, I applied policies at the OU level where possible, but for initial security hardening, I applied those to the entire domain. I did this because we want the whole domain protected, not just parts of it. 

***

### Inheritance, Enforcement, & Blocking

Inheritance, enforcement, and blocking are all very essential concepts for an AD admin to be aware of. 

- Inheritance: Policies get inherited or passed down through the OU hierarchy.
    - By default, policies applied to parents' OUs get inherited or passed down to child OUs.
    - Of course, the policies applied to less specific GPO link targets do get passed to the more specific ones that reside within, so long as the policies do not conflict. Inheritance, I believe moreso refers to the parent-to-child passing down of policy within the OU structure specifcally.
    
- Enforcement: GPO Links marked **Enforced** cannot be overridden by more specific policies.
    - Can be applied at the site, domain, or OU level.
    - Overrides blocking inheritance
    - In simple terms, a policy marked with 'Enforced' will always be applied

- Blocking: Blocking is used to block inheritance.
    - Child OUs will effectively ignore GPOs linked to the parent OU(s).
    - Good to use when we have a corporate-wide OU for all workstations. If we need one single sub OU to have different settings, blocking inheritance could be a great way to achieve that behavior.  

***

### Security and WMI Filtering

Both security and WMI filtering are ways to be more precise about what objects will actually get a given policy or policies applied to them. 


**Security Filtering:**

Security Filtering is an RBAC concept that is typically more related to security groups and access control than Group Policy. With that said, it can still help us narrow down exactly what objects we want a given policy to be applied to. 

Every GPO is just an object with an Access Control List(ACL). To apply a GPO, a user or computer must have the following two permissions:
- Read
- Apply Group Policy

By default, the Authenticated Users group (a securiyt group that is on all ADs by default) has both of these permissions.

Security filtering gives us the ability to change that ACL

So, for a given GPO, we can remove the Authenticated Users group from it, meaning just being in the Authenticated Users group is not sufficient to read or apply the policy. We can then select specific security groups that should be able to read and apply that policy. 

Depending on the environment, our OU structure could potentially perfectly mirror the reality of the business layout and necessity in terms of roles and the policy that would be applied. However, this is often not the case, especially in larger environments. Reality tends to be filled with exceptions to the rules, which is really where security filtering shines.

For example, say we want a GPO to apply to all workstations in our network besides like 3. If we want to only use OUs, we would have to create a brand new OU, move the three machines into that new OU, and could easily have to deal with inheritance-based issues or just generally some confusion. Alternatively, we could just remove those three machines from our workstation security group and modify that GPOs ACL to only target the workstation securiyt group, keeping the OU perfectly clean. 


**WMI Filtering:**

WMI filtering is essentially a way to apply GPOs using conditional logic based on technical criteria.

WIM filters are evaluated on the client at the time of policy processing, using current system data. 

What sorts of things can WMI filtering look for?
- OS Version
- OS Type (Workstation vs Server)
- Hardware (RAM, CPU, disk, etc)
- Installed software
- Domain role (Domain controller vs member server, etc)
- Type of computer (laptop or desktop)
- Services
- Network Adapters
- Virtualization (in some cases)
- so much more

To check out all of the specific information we can query, we can use the following PowerShell commands:

1. List all namespaces (**root\cimv2**, I believe, contains the huge majority of GPO WMI filters)
```powershell
Get-CimInstance -Namespace root -ClassName __Namespace |
Select Name
```

2. List all classes in a namespace (there will be lots in cimv2, again, replace that with other namespaces if desired)
```powershell
Get-CimClass -Namespace root\cimv2 |
Select CimClassName
```

3. Inspect the information within a specific class (of course, replace 'Win32_ComputerSystem' with the desired class)
```powershell
Get-CimInstance Win32_ComputerSystem | Format-List *
```

In GPMC, to apply WMI filtering, you add a new WMI filter, write your query (done in WQL, which looks very similar to SQL but is just very much simplified; knowing how to inspect classes as shown above is quite useful when it comes time to write the queries and you dont just want to copy filters from the internet), and then just link it to the specific GPO. 

We can also use PowerShell to validate our logic, largely using the same commands shown above, just with some where clauses or similar logic. For example:
```powershell
Get-CimInstance Win32_OperatingSystem |
Where-Object { $_.ProductType -eq 1 }
```
If we know that our machine has a ProductType of 1, we should get output from that command, indicating our machine would be returned by our filtering logic. If we do not get any output, that can tell us our logic may be off. 

***

### GPO Version Control 

Changing Group Policy is a big deal and can result in severe problems if done incorrectly. 

To hopefully avoid these major issues, we should always do the following (in a real enviornment of course)
- Ensure GPOs are backed up before any major change
- Test new policy on very small OUs, potentially make a few OUs for testing, even with just a single account inside, ensure these are placed within other OUs to see inheritance behavior if needed
- Slowly introduce changes; this makes things a lot easier to troubleshoot, as opposed to just implementing all changes at once

***

## Security Focused Group Policy

Within my lab environment, I experimented and learned about many common AD vulnerabilities involving:

- NTLM
- Kerberos
- SMB
- LDAP

Group Policy gives us a very easy way to control different aspects of these protocols that directly change the feasibility of attacks involving these protocols (among many others). 

Your Group Policy will directly affect your network's security posture, for better or for worse. I hope it's for the better, so please read on to see how Group Policy can increase your Active Directory network's security. 

**NTLM:**

NTLM is a legacy authentication protocol that has since been replaced, by default, with Kerberos. 

By default in a current AD envionrment, NTLM is enabled as a fallback option that is to be used only if Kerberos fails. 

NTLM has the following characteristics:
- Uses a challenge-response mechanism for authentication
- Widely considered insecure for a multitude of reasons
- Still commonly present within modern enviornments, usually to keep comptability with legacy systems that do not suppot Kerberos

NTLM Versions: 
- Lan Manager(LM) - This is kind of the predecessor to the NTLM we commonly see in networks today. It is extremely old and extremely insecure.
- NTLM - Also sometimes referred to as NTLMv1, this version still has pretty weak cryptography meaning the hashes can be easy to crack, it allows for relay and replay attacks, very insecure.
- NTLMv2 - Very similar but adds additional info including a client nonce, timestamp, target information, and uses stronger encryption making the hashes much harder to crack, the timestamps elimite the possibility of replay attacks, but relay is still viable.
- NTLMv2 with Session Security - Most secure option (still insecure), provides additional protections like message signing and optionally encryption using a derived session key, this protects against things like session hijacking and message tampering but does not elimitate credential theft or relay attacks. 

Group Policy NTLM Controls:
- Control general use of NTLM accross the AD enviornment:
    - Allowed: NTLM is allowed completely across the scope of the policy
    - Auditied: Allows admins to better monitor NTLM authentication in their network via increased logging
    - Disabled: NTLM is completely disabled accross the entire scope of the GPO
- Control the version of NTLM: There are a few different versions of NTLM that Windows supports which we can allow or disallow using Group Policy
    - Send LM & NTLM(v1): This is the most insecure setting possible and should always be avoided if possible. 
    - Send LM & NTLM(v1) - use NTLMv2 session secuirty if negotiated:  Uses NTLMv2 with session security if both machines support it but will still allow use of LM and NTLM, quite insecure still.
    - Send NTLM responses only: This only allows NTLMv1 to be used, again still quite insecure.
    - Send NTLMv2 responses only: This makes it default to NTLMv2, but allows LM and NTLM if a client does not support NTLMv2, again still quite insecure.
    - Send NTLMv2 responses only - refuse LM: Forces NTLMv2 and completely disallows the use of LM.
    - Send NTLMv2 responses only - refuse LM and NTLM: Forces only NTLMv2 to be used.
-  Restrict outgoing NTLM authentication - We can allow, audit, or disable outgoing NTLM traffic, we can also create an allow list of remote servers we want internal clients to be able to authenticate to using NTLM.
-  Restrict incoming NTLM authentication - We can allow, audit, or disable incoming NTLM traffic, we can also create an allow list of remote clients we want to be able to authenticate to internal servers using NTLM.

In my lab I intentionally allowed NTLM to practice performing and monitoring a wide array of exploits, I then learned how NTLM actually works internally, and how to secure a network against the assocaited risks of NTLM. 

**Kerberos:**

Kerberos is the default, modern authentication protocol used in Active Directory networks. 

Kerberos relies on the following:
- A central Key Distrobution Center(KDC):
    - The KDC is essentially two services the Authentication Service(AS) and the Ticket Granting Service(TGS).
    - Each domain has a one or more KDC(s) that serves as the cental authority for authentication within said doamin
- Ticket Granting Tickets(TGTs):
    - Ticket Granting Tickets are basically encrypted tokens that are issued by the KDC.
    - These tickets get issued to any identity after they sucessfully authenticate to the domain.
    - The TGS is used as a credential to avoid having to send passwords in plain text and allow users to access services without re-entering their password until the TGT expires.
- Service Tickets and Ticket Granting Service(TGS):
    - As mentioned above, the TGS is a service that runs as part of the KDC.
    - When a client wants to access a service, they send a request to the TGS containing their TGT, to prove they are authenticated, and the Service Principal Name(SPN) of the desired service.
    - If the request is valid, the TGS will issue a service ticket that allows access to that service for a limited time. 
- Shared Secretes Between Principals and the KDC:
    - Kerberos relies on symmetric encryption, meaning the client and server both encrypt and decypt with the same key.
    - If the client and server are able to decrypt the others message, they trust eachother, proving one another has access to the shared key.
    - This key is derrived on both sides from the clients password, meaning the keys and the plaintext passwords are never sent over the network.
    - Keys get stored on all domain controllers in a file called NTDS.dit.
    - There are often multiple different keys each derrived using different cryptographic algorithms on the users password.

Group Policy Kerberos Control:
- Maximum lifetime for a service ticket: Controls how long a service ticket is valid for. 
- Maximum lifttime for a ticket granting ticket: Controls how long a TGT is good for before renewing the ticket or re-entering credentials.
- Maximum lifetime for user ticket renewal: Controls how long a user can continually renew a TGT before being forced to renter credentials.
    - Renewal is a way to make a TGT valid for longer without re-entering credentials.
-  We can disable the Kerberos fallback to NTLM feature by just denying the use of NTLM as described earlier.
-  Configure encryption types used for Kerberos: These encryption methods are used to derive the keys. AES256 is the most secure, AES128 is also currently good, but if enforced could break older accounts until they reset their password, DES is the worst option and should be disabled, RC4 is the second worst and we should aim to reduce or elimite its usage.
-  Use gMSA where possible for service accounts to automatically rotate service account passwords and ensure all are long and random.
-  Mark admin accounts as sensitive and prevent delegation to reduce the risk of credential delegation and ticket forwarding
-  Audit TGT issuance and TGS requests to get lots of logs and better monitoring capability

I really spent a lot of time learning about these two foundational authetnication protocols present in Active Directory. Both have a huge amount of associated vulnerabilities and require real thought and understanding to properly defend against.




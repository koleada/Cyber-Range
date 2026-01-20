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








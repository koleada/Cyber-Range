### Group Policy (GPO) - Design, Implementation, & Security Controls

Group policy is one of the most powerful components of Active Directory. We touched on it briefly regarding the identity model, but here we will go a lot further.

In my cyber range AD enviornment, group policy is not just used as centalized configuration adn management console, it's also used as a primary security control plane that I becaame familiar with using to modify and secure authentication behavior, privleidge boundaries, network security, and attack surface. 

In this doc, I will detail how I learned and performed designing, structuring, and applying GPO within my lab enviornemnt specificalyl in by explaining the following:

1. Practice GPO design principles
2. Policy processing behavior and scope
3. Identity and authentication controls
4. Workstation and server configuration enforcement
5. Foundational security hardening using Group Policy

***

### Purpose of Group Policy in Corporate Active Directory Networks

From what I've learned Group Policy typically is used for the following functions:

- Enforece standardized system configuration requirrements accross an entire network of machines at once
- Define and enforce authentication and authorization requirements and behaviors within the AD envionrment
- Control code execution and system capabilities for specific Organizational Units, domains, etc
- Reduce attack surface within the network and apply baseline hardening strategies accross all domain devices
- Support compliance and auditing via logging and monitoring options and by having a centalzied console to view all policy applied within a domain

Within the AD network in my home lab, I practiced and learned about using group policy to mitigate or eliminate the possibility of various exploits, and to apply least privledge principles to different accounts accross the network. I also intentionally configured bad group policy for certain things to allow certain exploits to be performed for monitoring and research purposes. Additionally, I learned about foundational policy that I believe all AD enviornments should strive to have and inital hardening strategies within an AD enviornment. 

*** 

### Group Policy Hierarchy 

Before implementing Group Policy within a network, one must understand the processing order of Group Policy. By processing order I'm referring to the heirarchy of how Group Policy handles conflicting policies that are applied to a given subject (OU, doamin, etc). This is essential for troubleshooting Group Policy like when we are not sure why a specific policy is or is not being applied, or determining where inherited behavior is coming from, etc. 

Group Policy Objects are applied based on specificity, where the most specific rule takes precedent over the less specific ones. 

The order (from least specific to most) is as follows:

- Local Computer: There is a local (non-AD) GPO on all windows machines. This is completely outside of Active Directory, can only be managed by that one machine, and are stored locally. This is the least specific GPO.  
- Site: A site is a group of subnets typically used to represent a physical office location, so different locations get different sites. Sites have higher precedence than local and these are managed through AD.
- Domain: Domain wide policy gets applied to all users and computers in said domain. Some policies can only be applied at the domain level, these policies are located in *Default Domain Policy*, but adimns will still often apply policy outside of these domain wide. More specific than site wide and local computer policy.
- Organizational Unit: Organizational Units exist specifically for applying policy and delegation. They are essentially used to group accounts (Users, Machines, Services, etc) and apply policy to all accounts in that OU at once. There is heirarchy within OUs because OUs can reside within other OUs. The rule is that the 'deepest' OU gets precedent. So if we have a Users OU(parent) and within it a Sales OU(child), policies applied to the Sales OU would take precedent over those applied to the Users OU. OUs are the most specific GPO link targets.

The acronym to help remember this order is *LSDOU*. 

When a domain-joined computer or user logs in Windows first applies the local group policy, then site wide group policy, then domain group policy, then OU group policy beginning at the parent OU and moving through all child OUs. The deepest OU will have the most precedent by default. As Windows proceeds downward, if confliciting policy appears the more specific policy will override the less specific one. 

In my lab I applied policies at the OU level where possible, but for inital security hardening I applied those to the entire domain. I did this because we want the whole domain protected, not just parts of it. 

***

### Inheritance, Enforcement, & Blocking

Inheritance, enforcement, and blocking are all very essential concepts for an AD admin to be aware of. 

- Inheritance: Policies get inherited or passed down through the OU hierarchy.
    - By default, policies applied to parents OUs get inherited or passed down to child OUs.
    - Of course the policies applied to less specific GPO link targets, do get passed to the more specific ones that reside within so long as the policies do not conflict. Inheritance, I believe moreso refers to the parent-to-child passing down of policy within the OU structure specifcally.
    
- Enforcement: GPO Links marked **Enforced** cannot be overriden by more specific policies.
    - Can be applied at the site, domain, or OU level.
    - Overrides blocking inheritance
    - In simple terms, policy marked with 'Enforced' will always be applied

- Blocking: Blocking is used to block inheritance.
    - Child OUs will effecively ignore GPOs linked to the parent OU(s).
    - Good to use when we have corporate-wide OU for all workstations, if we need one single sub OU to have different settings, blocking inheritance could be a great way to achieve that behavior.  






















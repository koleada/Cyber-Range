# Domain & Forest Design 

The general design of the Active Directory environment was very simple. Again, this is all virtualzied and I only have so much RAM and storage, so I'm not looking to create multiple different domains and trusts right now. 

With that said, there were a couple of decisions I made to best represent a common enterprise deployment and to support realistic security testing and monitoring. Additionally, I do just want to lay out the basic design, general AD principles, relations between machines, and domain information so we can be on the same page in other writeups within this part of the repo. 

### Forest 

This lab just uses a single forest. 

The forest represents the broadest security boundary in Active Directory. Forests can encompass multiple domains and provide a centralized point to apply a general security policy to everything within the forest. Additionally, there is an inherent trust relationship between the domains in a forest. Enterprise admins are the only group that has full control over an entire forest. 

The single forest design was clearly the best choice, as it allows for additional domains if I ever wanted to create that in the future, and I really do not find it necessary to have multiple forests. Many enterprises may use only a single forest, so it definitely works for my purposes. 


### Domain Design

Within the forest, I just created a single domain called GOTHAM.local. 

A single-domain design really allows me to do everything I would want, like administration, monitoring endpoints, practicing security exploitation and defenses, etc. I do not see a reason to have additional domains here, because the admin related concepts it introduces are very minimal and it would mean having an entire new domain controller at a minimum, and potentially workstations too. Just from a pros and cons perspective, I do not think creating an entirely new windows server instance is worth it for a second domain, at least for my purposes. Again, lots of small-mid sized entierprises will use a single domain too. 

Multi-domain or child-domain designs, from my research, are typically implemented as a result of specific business or geographic requirements, which, again I did not find necessary to mimic here. 

**Purpose of the Domain in Active Directory:**
- Serves as the primary authentication and authorization boundary.
- Is another boundary for group policy application. When multiple domains are within a forest, we can apply more domain-wide policy in addition to forest-wide policy.
- Domains are a replication boundary for domain controllers. Each domain controller maintains its own copy of the data associated with Active Directory within a specific domain. In other words, domain controllers only store data for the specific domain they are on, if there are multiple domains, domain controllers can only belong to a single domain.

Within the domain we have our domain controller (Batman-DC), we have two workstations as well, Joker-PC and Bane-PC. (Yes, I did do a batman themed domain)

### Trust Relationships

No external or inter-domain trusts are used in this AD environment. 

Trust relationships can introduce additional attack paths, increase complexity, and need careful configuration and monitoring. Since we only use a single domain and forest, that removes the need for trusts. 

If I feel the need, I could always expand potentially by creating forest or extrernal trusts to simulate a business merger or aquisition, but for now I did not feel the need to do so. 


### Design Assumptions & Threat Model

My AD design (and most others) makes the following assumptions
1. Active Directory is the central identity/ authentication authority within the network
2. Compromise of a domain controller/ domain admin credentials equates to compromise of the entire network

This design allows security research to focus largely on the following concepts:
- Initial attack vectors, credential theft, and accessing user hashes
- Authentication abuse (kerberoasting, pass attacks, authentication coercion, NTLM authentication)
- Privilege escalation, once a credential is captured, how can we manage to increase the privileges we have to eventually achieve full domain compromise
- Detection and monitoring of all of these AD attacks


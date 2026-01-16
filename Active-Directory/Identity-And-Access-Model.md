# Identity & Access Model 

This document will describe what I, through doing my research, believe to be at least a good way to structure, manage, and assign privileges to identities within an Active Directory domain. 

Identity Design involves creating a structured framework for managing user, machine, and service accounts, and determining what resources each account should have access to. This is where we heavily rely on smart organization of domain accounts via the use of organizational units and groups to increase overall security, scalability, and efficient administration. 

For admins of Active Directory, this (along with GPO stuff), I believe, is the real meat and potatoes, meaning this is a foundational part of any AD admin's work, and also largely defines an organization's security posture 

***

### Organizational Units and Groups

**Organizational Units:**
Within my lab, I made specific Organizational Units (OUs) for various common departments in an office. 

I made groups like sales, IT, finance, and a couple of others. 

The significance of OUs is to organize domain accounts and to allow for very efficient application of standardized, baseline group policy to all people within a given OU at once.

Structuring users with OUs enables both fast enforcement of semi-targeted policy, and also a very clear separation of roles and access rights. OUs are also commonly used for delegation of admin rights. 

**Security Groups**
Additionally, I created Security Groups, which are used to control access to resources. In other words, groups are used in AD to implement *Role-Based Access Controls* (RBAC). 

**RBAC** - A role-based group (a group that represents a role) is given permission to access specific resources. Users who perform that role in real life are added to the role-based group.

So again, we add users to these security groups; their names will reference the OU names we created, or in some cases will have identical names. Unlike OUs, you cannot directly apply group policy to security groups. Instead, these groups are used to apply specific access controls. For example, we may only let users who belong to the Finance department access the file share for company financial documents. We may want users in the IT and Software Dev groups to be able to read and write to an internal code base, etc. 

**Differences:**
- Groups are *Security Principles* - they have an SID, they can appear on an ACL, they can be evaluated during authentication (users and computers are also security principles, this just gives a way to group them)
- OUs are NOT *Security Principles* - they do not have SIDs, they cannot belong to ACLs, and they also cannot be evaluated at logon
- The domain controller builds an *Access Token* whenever a user logs in to the domain. That token contains the user SID and the SIDs of ALL security groups that the user belongs to.
    - When the user goes to do something like access a file share, they will have to authenticate first, then the file server will analyze the resulting access token and determine if that user has the permissions necessary to access the share
- OUs are designed to be hierarchical, groups are designed to be flat

We want to be very strict and careful in regard to the groups and OUs in which we place any given user. Users should only be able to access what they explicitly need. 

**This is the main thing to keep in mind:** groups are for applying permissions and access controls, organizational units are for applying group policy. Both should be heavily used and well-maintained within your AD environment. 

***

### AGDLP Model 

This model is the generally recommended structure for security groups and permissions within AD. 

- A - Accounts, specific users
- G - Global security groups
- DL - Domain Local security groups
- P - Permissions, specific permissions or access rights

Global Groups - typically contain invidividual users, this type of group is typically used to describe who the users within it area, in other words, they usually map to specific roles or departments in an enterprise. A global group can belong to another global group too. 

Domain Local Groups - These define what access is granted; a good example name for a domain local group could be Financial_Files_RW for the permission to read and write financial documents. Permissions get assigned to these individual groups. 

Accounts -> Global Groups -> Domain Local Groups -> Permissions 

This methodology for structuring security groups and permissions makes for:
- clear separation of concerns
    - global user groups vs domain local groups
- very easy auditing
    - easy to determine why/how a user has access to a given resource
- makes applying access controls much more efficient, and promotes scalability

***

### General Identity Types

Identities in your typical AD environment can usually be categorized into at least one of these general groups:

- Standard user accounts
- Administrative accounts
- Service accounts
- Computer accounts

Each of these general groups is treated differently in terms of the privileges and access they have across the domain. 

***

### Standard User Accounts

These accounts represent your typical company employee, think like finance/accounting, sales, HR, and so forth.

Characteristics of accounts in this group:
- These accounts should not have administrative or any higher level priveldges
- Should only have a level of access that is needed to complete their day-to-day work, for example, they should be able to log in to the domain, access whatever internal applications or services they need, access the file shares they need to, etc
- Should be subject to group policy to enforce these security measures

Security Considerations & Assumptios:
- These accounts, in most cases, have the highest likelihood of being compromised by a phishing attack
  - Normal users are not as well-versed in spotting these attacks as someone in IT (usually)
  - Most domain users fall into this account
  - Email is used daily by most, if not all compnay employees
- Standard users should NEVER belong to privileged groups
- An attacker's initial entrance into the network will usually come from compromising one of these accounts   

***

### Administrative Accounts

Admin accounts are those that have permissions needed to manage the Active Directory domain and the systems joined to that domain. 

Security Principles:
- Admin privileges hand over the keys to the kingdom; a compromised admin account can easily lead to the destruction of your network
- Admin credentials should never be exposed to untrusted systems
- Separation between admin and non-admin identities is crucial

In a modern Active Directory environment, administrators will typically have a standard user account for their more mundane, everyday tasks, such as browsing the internet, emails, etc. They will also have one or more privileged accounts (potentially with different privilege levels) to perform the administrative tasks that require leveled privileges.  

For my home lab, I did not implement a fully teired administrative structure, but I certainly did create many elevated accounts with varying permissions, such as domain admin and local admins. 

I understand the tiered admin structure model, the risks associated with admin accounts, and also the difference from a securiyt standpoint between the different types of administrative accounts. 

***

### Service Accounts

Service accounts are specialized accounts used to run applications, services, or processes in a Windows environment. 

Since the services often need SYSTEM privledges, the service accounts associated with that service will also likely have elevated privledges. These privledges are necessary to the services to perform certain tasks on the system, and also to be able to use various AD specific resources/functionality. 

Security Considerations:
- Service accounts very commonly have elevated privileges (think processes that must be running as SYSTEM)
- Passwords are often static, dont have expiry, and potentially shared across accounts
- Compromise of a service account can often lead to lateral movement, privilege escalation, and total compromise of specific systems (compromising an SQL service account will likely give you full control over the database system)

In my lab, I just created and expieremented mostly with just normal services accounts. In production, though, it's likely a good idea to use *Managed Service Accounts** (MSAs). MASs automatically change service account passwords, automatically handle SPN registration, which ensures services can authenticate properly without manual config, and they allow for delegation for multiple services across machines. 

MSAs can save admins a lot of time if there are multiple service accounts in their directory, and they also significantly improve security. 

***

### Privledge Boundaries & Assumptions

A good AD identity model assumes the following:
- Standard user credentials will inevitably fall into the wrong hands eventually
- There could at any moment be an internal threat that already has credentials and/or network access
- Enough lateral movement will allow an intruder to escalate privileges and get the capability to do heavy damage to the environment

These assumptions remind IT professionals that following the basic concepts, such as least privilege, implementing strong group policy and access controls, clearly separating privileges, and implementing very strong monitoring and detection capability are so crucial for security. These concepts exist for a reason, intrusions always happen, hardening your envionrmetn and making it as hard as possible for an attacker to do anythign in the network, and mitigating damage are some of the best thigns you can do. Additionally, being able to detect intrusions and correlate anomalies can provide defenders a huge advantage. 



### Further Reading:
- [https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/implementing-least-privilege-administrative-models)

- [Lots of great OU information](https://theitbros.com/active-directory-organizational-unit-ou/)

- [Great article on how to protect privileged access](https://learn.microsoft.com/en-us/security/privileged-access-workstations/privileged-access-strategy)















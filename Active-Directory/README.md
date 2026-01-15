# Active Directory 

As I've mentioned many times in this repo, Active Directory is the centerpiece of this lab and the main reason I wanted to create this home lab. 

I'm not exactly sure what it is about AD, but I find it so interesting, like all aspects of it to, the management, troubleshooting, configuration, and, of course, the security side of things. I guess it just feels like such a technical marvel to me. Not to mention, I'm super interested in IT. I know Active Directory is so heavily used, and I want to prime myself to succeed in my career, which is very likely to involve AD. But yeah, just seeing the depth and complexity of Active Directory is so fascinating. Windows OS is quite complex and fascinating in itself, but that extra layer of domains and administration is so much fun to learn about and experiment with. 

***

### Goals

- Build a realistic, albeit minimized, enterprise-style Active Directory environment
- Understand and implement Active Directory security concepts and best practices, from exploit mitigation, anti-virus, least privilege and ACLs, group policy, secure protocols, etc
- Have the ability to perform AD exploits and implement detection and mitigation for said exploits
- Understand how administration and configuration affect the security of an Active Directory environment

***

### Scope 

In this folder, we will focus on the following concepts:

1. Domain and controller design and setup
2. User and group management
3. Group policy strategy (security-focused)
4. Authentication protocols 
5. PKI and Active Directory Certificate Services
6. Hardening and detection considerations

*** 

### Basic Active Directory Design & Security Philosophy 

When designing any network, but especially an AD environment, I found these to be the main guiding principles in keeping the network secure/ 

1. Internal networks can never be inherently trusted
2. Credentials will eventually be exposed, there will be an intrusion into the network, it's just a matter of when
3. Detection is equally important to protection/prevention

Thinking in this way really helped me to secure every part and practice defense-in-depth. We want to be thinking about damage mitigation and trying to implement security checks in as many places as possible to make it as hard as possible for an attacker to move through the network and execute known exploits. 






# Group Policy for Security

Within my lab environment, I experimented and learned about many common AD vulnerabilities involving:

- NTLM
- Kerberos
- SMB
- LDAP

Group Policy gives us a very easy way to control different aspects of these protocols that directly change the feasibility of attacks involving these protocols (among many others). 

Your Group Policy will directly affect your network's security posture, for better or for worse. I hope it's for the better, so please read on to see how Group Policy can increase your Active Directory network's security. 

***

### NTLM

NTLM is a legacy authentication protocol that has since been replaced, by default, with Kerberos. 

By default in a current AD envionrment, NTLM is enabled as a fallback option that is to be used only if Kerberos fails. 

NTLM has the following characteristics:
- Uses a challenge-response mechanism for authentication
- Widely considered insecure for a multitude of reasons
- Still commonly present within modern enviornments, usually to keep comptability with legacy systems that do not suppot Kerberos

**NTLM Authentication Protocol Diagram**
![A daigram of the NTLM Authentication Protocol](https://github.com/koleada/Cyber-Range/blob/main/Diagrams/NTLM.drawio.png?raw=true)

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

***

### Kerberos:

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
 
**Kerberos Authentication Protocol Diagram**
![A daigram of the Kerberos Authentication Protocol](https://raw.githubusercontent.com/koleada/Cyber-Range/refs/heads/main/Diagrams/Kerberos%20Protocol%20Diagram.png)

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

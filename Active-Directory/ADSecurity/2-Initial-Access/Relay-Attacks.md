# Relay Attacks

Relay attacks are an authentication forwarding attack that allows us to relay hashes we captured to other machines. 

We will focus on SMB relay, but this can be done with other protocols too. 

This can be used to authenticate into a machine and even get a shell out of it, without even having an actual clear-text password. The resulting shell could be used to explore the file system, add a new local admin for persistance, and much more. 

### Why Does SMB Relay Exist

- NTLM is challenge-response-based, not mutual authentication, where both parties are in agreement
- NTLM does not cryptographically ensure that the authentication goes to a specific place
- SMB, at least in older versions, did not require message signing

### NTLM Refresher

We discussed this in detail in the LLMNR Poisoning overview, so this will be quick.

NTLM is a 3-message exchange:
1. Negotiate - client initiates the NTLM flow by negotiating the NTLM options with the server
2. Challenge - server responds with a challenge
3. Authentication - client responds with hashed password + challenge

### SMB Relay Flow

1. We begin by putting ourselves in the position to get a hash, which can be done with LLMNR poisoning, IPv6 DNS takeover, Printer authentication, authentication coercion, etc
2. The victim machine initiates the NTLM authentication flow by sending the negotiate message
3. Attacker machine opens a connection to the machine we want to relay to, and only after that responds with the challenge
4. Victim responds with the hashed password and challenge
5. Attacker forwards the provided username, domain name, and hash to the target machine
6. The target machine validates the credentials and permits access to the attacker if SMB signing is not required and NTLM is allowed

Once on the target machine, the attacker can:
  - Execute SMB operations, add honeypot files, or social engineering type stuff
  - Modify existing files
  - Execute commands (if privileges are sufficient)
  - Choose to relay to other protocols (LDAP, HTTP, AD CS) if SMB is not

### LDAP Relay 

LDAP Relay can often lead to more severe exploitation than SMB Relay. In the modern landscape, it seems a lot of people prefer to relay to LDAP/LDAPS and AD CS. 

Again, LDAP signing must not be required to make relay possible. 

This type of relay could allow us to add certain users to certain groups, modify access control lists, and possible most notably, abuse AD CS. 

This is a very common way to begin with low-privilege credentials, but quickly escalate to some sort of admin. 

### Relay vs Pass-the-hash

- Relay attacks are prevented with signing, and  pass-the-hash is prevented by disabling NTLM
- Relay attacks do not require us to ever see hashes, its realtime, pass-the-hash allows us to attempt to use old hashes instead of a real-time NTLM authetnication 

### Defenses and Mitigations

1. Signing
  - This is the biggest requirement for preventing relay attacks
  - What is signing: it ensures that all messages are cryptographically signed and bound to a session key, the receiver can ensure the message was not modified and that it actually came from the place it says it did
  - Traffic cannot be signed unless you have the session key, which is derived from NTLM or Kerberos authentication
  - Signing ensures the integrity of the messages
  - Must *require* signing against other important protocols like LDAP and HTTP
  - Signing is required by default on modern Windows machines; legacy systems will often still have it just *enabled,* which will not prevent this attack

2. Disable LLMNR and NBT-NS, as this can often prevent hashes from being obtained
3. Follow least privilege by only allowing SMB access to those who need it
4. Use Kerberos whenever possible 


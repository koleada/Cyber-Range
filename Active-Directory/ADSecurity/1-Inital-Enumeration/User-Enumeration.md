# User Enumeration 

Knowing the usernames of domain users is very important and can help us to gain our initial foothold into the network 

There are a few different ways to attempt to find user information without credentials

1. Username Enumeration via SMB
  - `nxc smb <dc_ip> -u '' -p '' --users`
  - Again were using the netexec SMB tooling, but this time to enumerate users, try guest as the username too
  - We try to establish an anonymous (or guest) SMB session, and then try to bind to SAMR and/or LSARPC to enumerate users
    - Different from the RPC bind technique because this one relies on the SMB session being established 
    
2. RID Bruteforce
  - `nxc smb <dc_ip> --rid-brute 10000 # bruteforcing RID`
  - This time were using a brute force approach again, facilitated by netexec
  - RID = Relative Identifier - Different RIDs map to different user groups like Administrator, Guest, Domain Admins, etc
  - Basically, the RID is part of the Security Identifier, which is a long string formatted as (domain SID + RID)
  - So what this is doing is finding the domain SID, which is trivial, then looping through all RIDs in the number range, since they are a predictable value
  - When we request a valid user SID, the DC will respond with the username it corresponds to
  - This is not likely to work in modern AD enviornments but it's still a cool technique

3. Anonymous RPC Bind
  -  `net rpc group members 'Domain Users' -W '<domain> -l <ip> -U '%'`
  -  Basically, we are trying to get an unauthenticated RPC connection and from there just enumerating the *Domain Users* group
  -  We use the domain controller IP here
  -  Again, rare to find, but still could appear

4. Kerberos User Enumeration
  - `nmap -p 88 --script=krb5-enum-users --script-args="krb5-enum-users.realm='<domain>',userdb=<user_list_file>" <dc_ip>`
  - `kerbrute userenum -d <domain> <userlist>`
  - Here we are using an aspect of Kerberos where we get different response messages based on whether we supply a valid username or not, which we use to detect if we found a valid user
  - Needs a wordlist, can always try to make custom, targeted wordlists, potentially using company email schema, common AD usernames, etc


In all honesty, I would not be starting off with any of these techniques. Knowing usernames can certainly be useful, but how much this will help actually gain an initial foothold is debatable. Either way, these are important techniques and can be useful. 


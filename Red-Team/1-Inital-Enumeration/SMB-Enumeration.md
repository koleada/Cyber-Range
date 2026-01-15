# SMB Enumeration 

This is a very important part of the initial enumeration

Very early in approaching an AD envionrment we should be attempting to find shares accessible to an unauthenticated user on the machines we have access to

### SYSVOL 

SYSVOL is an SMB share replicated to all DCs and readable by all authenticated users. 

- Old GGP XML files like Groups.xml, SchedulesTasks.xml may have *cPassword* fields
-  These are AES encrypted with a hardcoded Microsoft key that's public, so we can decrypt them offline
-  This was patched a while ago, but unless someone went in and deleted the old files that were automatically created, they will still exist on a machine
-  Unlikely that these are accessible while being unauthenticated, but still worth attempting
-  `smbclient //dc-ip/SYSVOL -U ''` - Enumerates the SYSVOL shares, also try with the username 'guest'
  - `grep -r cPassword`

### Public Shares

Even in non-SYSVOL shares, we can still potentially find some very useful information. 

Finding PII in a public share is still a good finding that is more than worth including in a penetration testing report 

### Commands:

1. `smbclient -L //<dc-ip> -U 'guest'` - Enumerates guest accessible shares on the domain controller (replace with IP of other machines too)
2. `smbclient -L //<dc-ip> -U ''` - Enumerates shares that can be accessed anonymously on the domain controller
3. `nmap --script smb-enum-shares -p 139,445 target` - Better for enumerating SMB shares across many machines
4. Can also use netexec to do this too: `nxc smb <ip_range> -u '' -p ''`


# LDAP Enumeration 

We can access really important info if we are able to do anonymous binds via LDAP (rare but still seen, so we should be checking)

### Quick and Easy Ways of Testing for Anonymous Bind

1. Impacket's ldapdomaindump will dump info if an anonymous bind is possible in our environment
2. LDAPSearch tool with -x (anon bind) and -H ldap://<target>
  - `ldapsearch -x -H <dc_ip> -s base`
  - `ldapsearch -x -H ldap://<dc-ip> -b "CN=Enrollment Services,CN=Public Key Services,CN=Services,CN=Configuration,DC=domain,DC=com" "(objectClass=pKIEnrollmentService)" dn`
    - Tells us if there is a CA in the AD environment, really good information to know, so we can avoid wasting time with certificates if there is no CA  
3. `nmap -n -sV --script 'ldap*' and not brute -p 389 <dc_ip>` - another good way to enumerate LDAP and try anonymous bind

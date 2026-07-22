

# Forest — HTB Write-up

**Difficulty:** Easy
**OS:** Windows
**Tags:**  Active Directory, AS-REP Roasting, User enumeration

## Summary
Windows box that does not provide any credentials. SMB enumeration doesn't provide any information, but knowledge of Active Directory will help gain a foothold through AS-REP roasting. Once on the machine, Windows privilege escalation meets a dead end, but reviewing ACL's provides a chain to domain compromise. 

## Reconnaissance
- Nmap scan
![nmap](forest-1.png)
- Time skew present, so any Kerberos will need to update time to match domain controller
![clock](forest-2.png)

## Enumeration
- Starting with SMB
![smb](forest-3.png)
- Although anonymous connection is allowed, there are no shares to read
![anonymous](forest-4.png)
- Checking for users with crackmapexec (can also use NetExec) and locating potential user list
![users](forest-5.png)

## Initial Foothold
- With a list of users, we can use several tools to proceed.
![users2](forest-6.png)
- First we check for anonymous LDAP (pre-auth not required) or also known as AS-REP Roasting
- We get a hash of the svc-alfresco user
![hash](forest-6.png)
- Using hashcat to crack and get the plain text password
![hashcat](forest-7.png)
- User is able to remote access the box
![remtoe](forest-8.png)

## Privilege Escalation
- Normal privilege escalation produced no results, so we run Bloodhound and locate an ACL chain that will get us to domain admin
![bloodhound](forest-9.png)
- First we create a new user
![users3](forest-10.png)
- Then we add our new user to the Exchange Windows Permissions group
![newuser](forest-11.png)
- From here, there are several options with WriteDacl, but the most straight forward is to use DCSync attack
- We first download PowerView.ps1 and setup are variables
![dcsync](forest-12.png)
- Then we run the commands to grant DCSync access to our user we created
![powerview](forest-13.png)
- Then we use Impacket's secretsdump tool to get the hash of the administrator
![cmd](forest-14.png)
- With the hash obtained, we use Impacket's psexec tool to remote onto the box with system level access
![psexec](forest-15.png)

## Key Takeaways
- Active Directory knowledge is key as we do not start with any credentials
- CrackMapExec/NetExec along with Impacket are essential tools for windows machines
	- BloodyAD and Certipy could also be used in this situation

---
*Flags redacted per HTB write-up guidelines. This write-up covers retired content only.*

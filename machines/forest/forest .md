

# Forest — HTB Write-up

**Difficulty:** Easy
**OS:** Windows
**Tags:**  Active Directory, AS-REP Roasting, User enumeration

## Summary
Windows box that does not provide any credentials. SMB enumeration doesn't provide any information, but knowledge of Active Directory will help gain a foothold through AS-REP roasting. Once on the machine, Windows privilege escalation meets a dead end, but reviewing ACL's provides a chain to domain compromise. 

## Reconnaissance
- Nmap scan 
- Time skew present, so any Kerberos will need to update time to match domain controller
## Enumeration
- Starting with SMB
- Although anonymous connection is allowed, there are no shares to read
- Checking for users with crackmapexec (can also use NetExec) and locating potential user list

## Initial Foothold
- With a list of users, we can use several tools to proceed. 
- First we check for anonymous LDAP (pre-auth not required) or also known as AS-REP Roasting
- We get a hash of the svc-alfresco user
- Using hashcat to crack and get the plain text password
- User is able to remote access the box


## Privilege Escalation
- Normal privilege escalation produced no results, so we run Bloodhound and locate an ACL chain that will get us to domain admin
- First we create a new user
- Then we add our new user to the Exchange Windows Permissions group
- From here, there are several options with WriteDacl, but the most straight forward is to use DCSync attack
- We first download PowerView.ps1 and setup are variables
- Then we run the commands to grant DCSync access to our user we created
- Then we use Impacket's secretsdump tool to get the hash of the administrator 
- With the hash obtained, we use Impacket's psexec tool to remote onto the box with system level access

## Key Takeaways
- Active Directory knowledge is key as we do not start with any credentials
- CrackMapExec/NetExec along with Impacket are essential tools for windows machines
	- BloodyAD and Certipy could also be used in this situation

---
*Flags redacted per HTB write-up guidelines. This write-up covers retired content only.*
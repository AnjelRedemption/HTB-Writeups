# Cicada — HTB Write-up

**Difficulty:** Easy
**OS:** Windows
**Tags:** Active Directory, Enumeration

## Summary
SMB share leaked credentials for unknown user. Created potential user list and spraying to locate another user with password. This user has access to another share with additional leaked credentials and remote access. Once on the box, the user has SeBackupPrivilege to capture SAM information and gain Administrator access.

## Reconnaissance
- Nmap scan shows Windows device and standard Active Directory ports
- SMB is open, this will be our first point of Enumeration
![Nmap scan](images/Nmap_Scan.png)

## Enumeration
- Accessing SMB using smbclient and anonymous auth allows access
- File located under HR folder
 ![SMB-HR](images/SMB_HR.png)
- Reading the contents provides a potential password
![new-hire](images/new_hire.png)
## Initial Foothold
- First a list of potential users is created
- Using Impacket's tool "lookupsid" and guest authentication we are able to create a user list
![impacket](images/SMB_HR.png)
![[Pasted image 20260721123539.png]]
- Using the userlist we created and NetExec to spray for potential users and locating michael.wrightson with the leaked credential
![[Pasted image 20260721123726.png]]
- Using this new user to further enumerate the users confirms password located in description field
![[Pasted image 20260721124813.png]]
- Checking this users access confirms they can read the Dev share
![[Pasted image 20260721125019.png]]
- Accessing Dev as david user and locating a Powershell script
![[Pasted image 20260721125104.png]]
- Reading the script exposed plain text credentials for emily.oscars
![[Pasted image 20260721125242.png]]
- Confirmed this user has Remote Management access
- Using Evil-Winrm to gain shell access
![[Pasted image 20260721125354.png]]

## Privilege Escalation
- Once initial access was gained, user enumeration shows SeBackupPrivilege is enabled
![[Pasted image 20260721125443.png]]
- This privilege allows users to perform backup operations, including registry backups of SAM/SECURITY/SYSTEM
- Running the backup commands and exporting them
![[Pasted image 20260721125657.png]]
- using Impacket's secretsdump tool to get hash of the administrator
![[Pasted image 20260721125807.png]]
- Testing with Evil-winrm confirms access and root flag
![[Pasted image 20260721125915.png]]
## Key Takeaways
- Enumeration of potential AD users is key
	- Additional Tools that could work are Kerbrute or Enum4Linux

---
*Flags redacted per HTB write-up guidelines. This write-up covers retired content only.*

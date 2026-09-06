# Cicada — Hack The Box

| Attribute | Details |
|---|---|
| Difficulty | Easy |
| OS | Windows |
| Status | Retired |
| Focus | SMB enumeration, credential exposure, password spraying, WinRM, SeBackupPrivilege |

## Overview

Cicada demonstrates a Windows Active Directory compromise driven primarily by
poor credential management and excessive exposure of sensitive information.
Anonymous SMB access reveals an onboarding password, while domain user
enumeration provides valid usernames for controlled password spraying.
Additional credentials are exposed through an Active Directory user description
and a PowerShell script stored on a network share. The final user possesses
`SeBackupPrivilege`, allowing the SAM and SYSTEM registry hives to be copied
and the local Administrator NTLM hash to be recovered.

## Attack Path

`Anonymous SMB → Onboarding Password → User Enumeration → Password Spray → Credential Exposure → DEV Share → WinRM → SeBackupPrivilege → Administrator`

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
![impacket](images/loookupsid.png)
![impacket](images/loookupsid2.png)
- Using the userlist we created and NetExec to spray for potential users and locating michael.wrightson with the leaked credential
![userlist](images/user_spraying.png)
- Using this new user to further enumerate the users confirms password located in description field
![discription](images/nxc_users.png)
- Checking this users access confirms they can read the Dev share
![dev](images/SMB_dev.png)
- Accessing Dev as david user and locating a Powershell script
![script](images/smb_dev.png)
- Reading the script exposed plain text credentials for emily.oscars
![script](images/powershell_script.png)
- Confirmed this user has Remote Management access
- Using Evil-Winrm to gain shell access
![evil-winrm](images/evil-winrm-cicada.png)

## Privilege Escalation
- Once initial access was gained, user enumeration shows SeBackupPrivilege is enabled
![priv-sesc](images/priv-esc.png)
- This privilege allows users to perform backup operations, including registry backups of SAM/SECURITY/SYSTEM
- Running the backup commands and exporting them
![sam](images/sam-sav.png)
- using Impacket's secretsdump tool to get hash of the administrator
![secrets](images/secretsdump.png)
- Testing with Evil-winrm confirms access and root flag
![root](images/root.png)
## Key Takeaways
- Enumeration of potential AD users is key
	- Additional Tools that could work are Kerbrute or Enum4Linux

---
*Flags redacted per HTB write-up guidelines. This write-up covers retired content only.*

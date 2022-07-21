## Active Directory Recon

### Recon from non-Domain-Joined Windows Computer
Note:
Configure your system DNS server to be the IP address of a domain controller in the target domain firstly. 
After the "runas" command, you must check to access SYSVOL and NETLOGON folders with the following command: 
```powershell
net view \\unsafe.local\
```
You must see the SYSVOL and NETLOGON folders if you supply a valid password for the "runas" command.
Information for the following examples:
```text
Domain DNS name: unsafe.local
Domain NetBIOS name: UNSAFE
Domain username: ruser
Child domain DNS name: gotham.unsafe.local
Child domain NetBIOS name: GOTHAM
```

- **Nslookup (for finding DCs)**
```powershell
#Open cmd
C:\> nslookup
   > set type=SRV
   > _ldap._tcp.dc._msdcs.unsafe.local 
```

- **SharpHound**
```powershell
#Method 1

#1. Spawn a CMD shell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser cmd.exe

#2. Run SharpHound, using the -d flag to specify the AD domain you want to collect information from. You can also use any other flags you wish.
C:\> SharpHound.exe -d unsafe.local -c All --outputdirectory C:\Users\desktop2\Desktop


#Method 2
C:\> SharpHound.exe -d unsafe.local -c All --ldapusername ruser --ldappassword Password
```

- **PowerView**
```powershell
#1. Spawn a Powershell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser powershell.exe

#2. Set Execution policy as Bypass
Set-ExecutionPolicy Bypass -Scope CurrentUser

#3. Import Module
Import-Module C:\Users\desktop2\Desktop\AD-Tools\Tools\PowerView_dev.ps1

#4. Running cmdlet
Get-NetDomain
```

- **RSAT**
```powershell
1. Download and install RSAT
2. Run cmd.exe as Administrator
3. Spawn a MMC as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser "mmc /server=unsafe.local"
4. File > Open > File name: C:\Windows\System32 > dsa (for example) > click
```

- **PurpleKnight**
```powershell
#1. Download PurpleKnight and unzip the archive
#2. Spawn a CMD shell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser cmd.exe
#3. Set Execution policy as Bypass
C:\> powershell -c "Set-ExecutionPolicy Bypass -Scope CurrentUser"
#4. Run the executable from CMD
C:\> .\PurpleKnight.exe
#5. It will be opened and not detect a forest as expected. Type the domain name (e.g: unsafe.local) and click select > next > 'run tests'.
```

- **ADACLScanner (unstable)**
```powershell
#1. Spawn a Powershell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser powershell.exe

#2. Set Execution policy as Bypass
 Set-ExecutionPolicy Bypass -Scope CurrentUser

#3. Generate a report from the command line:
 .\ADACLScan.ps1 -Base "DC=unsafe,DC=local" -Scope subtree -Server dc.unsafe.local -Port 389 -Output HTML -Show
 .\ADACLScan.ps1 -Base "DC=unsafe,DC=local" -Scope subtree -Server dc.unsafe.local -Port 389 -EffectiveRightsPrincipal ruser -Output HTML -Show
```

- **Pingcastle**
```powershell
#1. Download Pingcastle and unzip the archive
#2. Spawn a CMD shell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser cmd.exe
#3. Generate a HTML healthcheck report for domain:
C:\> .\PingCastle.exe --log --healthcheck --server unsafe.local
#4. To scan for the Zerologon vulnerability:
C:\> .\PingCastle.exe --log --scanner zerologon --server unsafe.local
```

- **.Net SDS.AD namespace**
```powershell
#1. Spawn a Powershell as a user in that domain using runas and its /netonly flag and enter the password.
runas /netonly /user:UNSAFE\ruser powershell.exe

#Get the forest information:
 [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest()

#Get the current user's domain information:
 [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

#Get information of DCs:
 [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().DomainControllers

#Find Primary DC:
 [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain().pdcroleowner

#Get trusts for current domain:
 ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()

#Get a list of sites in the forest:
 [System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().sites

#Determine the SID filtering status of a trust. If the output is "true", SID filtering is enabled.
 $domain="gotham.unsafe.local"
 ([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetSidFilteringStatus($domain)
```

### Reference
https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound.html

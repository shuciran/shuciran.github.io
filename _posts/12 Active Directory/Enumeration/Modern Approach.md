There are several more modern tools capable of enumerating AD environments. PowerShell _cmdlets_ like _Get-ADUser_ work well but they are only installed by default on domain controllers (as part of RSAT), and while they may be installed on Windows workstations from Windows 7 and up, they require administrative privileges to use.

We can, however, use PowerShell (the preferred administration scripting language for Windows) to enumerate AD. In this section, we will develop a script that will enumerate the AD users along with all the properties of those user accounts.

Although this is not as simple as running a command like _net.exe_, the script will be quite flexible, allowing us to add features and functions as needed. As we build the script, we will discuss many technical details relevant to the task at hand. Once the script is complete, we can copy and paste it for use during an assessment.

As an overview, this script will query the network for the name of the Primary domain controller emulator and the domain, search Active Directory and filter the output to display user accounts, and then clean up the output for readability.

A Primary domain controller emulator is one of the five operations master roles or _FSMO_ roles performed by domain controllers. Technically speaking, the property is called _PdcRoleOwner_ and the domain controller with this property will always have the most updated information about user login and authentication.

This script relies on a few components. Specifically, we will use a _DirectorySearcher_ object to query Active Directory using the _Lightweight Directory Access Protocol_ (LDAP), which is a network protocol understood by domain controllers also used for communication with third-party applications.

LDAP is an _Active Directory Service Interfaces_ (ADSI) provider (essentially an API) that supports search functionality against an Active Directory. This will allow us to interface with the domain controller using PowerShell and extract non-privileged information about the objects in the domain.

Our script will center around a very specific _LDAP provider path_ that will serve as input to the _DirectorySearcher_ .NET class. The path's prototype looks like this:

```
LDAP://HostName[:PortNumber][/DistinguishedName]
```

To create this path, we need the target _hostname_ (which in this case is the name of the domain controller) and the _DistinguishedName_ (DN) of the domain, which has a specific naming standard based on specific Domain Components (DC).

First, let's discover the hostname of the domain controller and the components of the DistinguishedName using a PowerShell command.

Specifically, we will use the _Domain_ class of the _System.DirectoryServices.ActiveDirectory_ namespace. The _Domain_ class contains a method called _GetCurrentDomain_,[10](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/active-directory-attacks/active-directory-enumeration/a-modern-approach#fn10) which retrieves the _Domain_ object for the currently logged in user.

Invocation of the _GetCurrentDomain_ method and its output is displayed in the listing below:

```
PS C:\Users\offsec.CORP> [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()


Forest                  : corp.com
DomainControllers       : {DC01.corp.com}
Children                : {}
DomainMode              : Unknown
DomainModeLevel         : 7
Parent                  : 
PdcRoleOwner            : DC01.corp.com
RidRoleOwner            : DC01.corp.com
InfrastructureRoleOwner : DC01.corp.com
Name                    : corp.com
```

According to this output, the domain name is "corp.com" (from the _Name_ property) and the primary domain controller name is "DC01.corp.com" (from the _PdcRoleOwner_ property).

We can use this information to programmatically build the LDAP provider path. Let's include the _Name_ and _PdcRoleOwner_ properties in a simple PowerShell script that builds the provider path:

```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"

$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$SearchString
```

```powershell
In this script, _$domainObj_ will store the entire domain object, _$PDC_ will store the _Name_ of the PDC, and _$SearchString_ will build the provider path for output. Notice that the _DistinguishedName_ will consist of our domain name ('corp.com') broken down into individual domain components (DC), making the DistinguishedName "DC=corp,DC=com" as shown in the script's output:
```

```powershell
LDAP://DC01.corp.com/DC=corp,DC=com
```

This is the full LDAP provider path needed to perform LDAP queries against the domain controller.

We can now instantiate the _DirectorySearcher_ class with the LDAP provider path. To use the _DirectorySearcher_ class, we have to specify a _SearchRoot_, which is the node in the Active Directory hierarchy where searches start.

The search root takes the form of an object instantiated from the _DirectoryEntry_ class. When no arguments are passed to the constructor, the _SearchRoot_ will indicate that every search should return results from the entire Active Directory. The code in below shows the relevant part of the script to accomplish this.

```powershell
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"

$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry($SearchString, "corp.com\offsec", "lab")

$Searcher.SearchRoot = $objDomain
```

With our _DirectorySearcher_ object ready, we can perform a search. However, without any filters, we would receive all objects in the entire domain.

One way to set up a filter is through the _samAccountType_ attribute, which is an attribute that all user, computer, and group objects have. Please refer to the linked reference for more examples, but in our case we can supply 0x30000000 (decimal 805306368) to the filter property to enumerate all users in the domain, as shown in Listing 10:

```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"

$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry

$Searcher.SearchRoot = $objDomain

$Searcher.filter="samAccountType=805306368"

$Searcher.FindAll()
```

We have added the samAccountType filter through the _.filter_ property of our _$Searcher_ object and then invoked the _FindAll_ method to conduct a search and find all results given the configured filter.

When run, this script should enumerate all the users in the domain:

```
Path                                                                 Properties                                   
----                                                                 ----------                                   
LDAP://CN=Administrator,CN=Users,DC=corp,DC=com                      {admincount...
LDAP://CN=Guest,CN=Users,DC=corp,DC=com                              {iscritical...
LDAP://CN=DefaultAccount,CN=Users,DC=corp,DC=com                     {iscritical...
LDAP://CN=krbtgt,CN=Users,DC=corp,DC=com                             {msds-...
LDAP://CN=Offsec,OU=Admins,OU=CorpUsers,DC=corp,DC=com               {givenname...
LDAP://CN=Jeff_Admin,OU=Admins,OU=CorpUsers,DC=corp,DC=com           {givenname...
LDAP://CN=Adam,OU=Normal,OU=CorpUsers,DC=corp,DC=com                 {givenname...
LDAP://CN=iis_service,OU=ServiceAccounts,OU=CorpUsers,DC=corp,DC=com {givenname...
LDAP://CN=sql_service,OU=ServiceAccounts,OU=CorpUsers,DC=corp,DC=com {givenname...
```

This is good information but we should clean it up a bit. Since the attributes of a user object are stored within the _Properties_ field, we can implement a double loop that will print each property on its own line.

The complete PowerShell script will collect all users along with their attributes:

```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()

$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"

$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

$Searcher = New-Object System.DirectoryServices.DirectorySearcher([ADSI]$SearchString)

$objDomain = New-Object System.DirectoryServices.DirectoryEntry

$Searcher.SearchRoot = $objDomain

$Searcher.filter="samAccountType=805306368"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    Foreach($prop in $obj.Properties)
    {
        $prop
    }
    
    Write-Host "------------------------"
}
```

The retrieved information can be quite overwhelming since user objects have many attributes. The listing below shows a partial view of the Jeff_Admin user's attributes.

```
givenname               {Jeff_Admin}
samaccountname          {jeff_admin}
cn                      {Jeff_Admin}
pwdlastset              {131623291900859206}
whencreated             {05/02/2018 18.33.10}
badpwdcount             {0}
displayname             {Jeff_Admin}
lastlogon               {0}
samaccounttype          {805306368}
countrycode             {0}
objectguid              {130 114 89 75 220 233 3 76 170 206 193 232 122 112 176 32}
usnchanged              {12938}
whenchanged             {05/02/2018 19.20.52}
name                    {Jeff_Admin}
objectsid               {1 5 0 0 0 0 0 5 21 0 0 0 195 240 137 95 239 58 38 166 116 233
logoncount              {0}
badpasswordtime         {0}
accountexpires          {9223372036854775807}
primarygroupid          {513}
objectcategory          {CN=Person,CN=Schema,CN=Configuration,DC=corp,DC=com}
userprincipalname       {jeff_admin@corp.com}
useraccountcontrol      {66048}
admincount              {1}
dscorepropagationdata   {05/02/2018 19.20.52, 01/01/1601 00.00.00}
distinguishedname       {CN=Jeff_Admin,OU=Admins,OU=CorpUsers,DC=corp,DC=com}
objectclass             {top, person, organizationalPerson, user}
usncreated              {12879}
memberof                {CN=Domain Admins,CN=Users,DC=corp,DC=com}
adspath                 {LDAP://CN=Jeff_Admin,OU=Admins,OU=CorpUsers,DC=corp,DC=com}
...
```

According to the output above, the Jeff_Admin account is a member of the Domain Admins group. Using our _DirectorySearcher_ object, we could use a filter to locate members of specific groups like Domain Admin, or use a filter to specifically search only for the Jeff_Admin user.

In the filter property, we can set any attribute of the object type we desire. For example, we can use the _name_ property to create a filter for the Jeff_Admin user as shown below:

```
$Searcher.filter="name=Jeff_Admin"
```

Although this script may seem daunting at first, it is extremely flexible and can be modified to assist with other AD enumeration tasks.
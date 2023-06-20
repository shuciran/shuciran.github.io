Installation is easier with the standalone image since it has already packed all that is needed to work in any system all we need to do is download [Volatility](https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf) 

### Filesystem:
```bash
./volatility_2.6_lin64_standalone -f <IMAGE> imageinfo
```
### Processes List
Print all running processes by following the EPROCESS lists
```bash
./volatility_2.6_lin64_standalone -f ~/Documents/HTB/Challenges/Remniscient/reminiscent/flounder-pc-memdump.elf --profile=Win7SP1x64 dumpfiles -Q 0x000000001e8feb70 --name file -D ~/Documents/HTB/Challenges/Remniscient
```
### Command Line
Display process command-line arguments
```bash
./volatility_2.6_lin64_standalone -f ~/Documents/HTB/Challenges/Remniscient/reminiscent/flounder-pc-memdump.elf --profile=Win7SP1x64 cmdline
```
## Searching file
```bash
./volatility_2.6_lin64_standalone -f ~/Documents/HTB/Challenges/Remniscient/reminiscent/flounder-pc-memdump.elf --profile=Win7SP1x64 filescan | grep resume
```
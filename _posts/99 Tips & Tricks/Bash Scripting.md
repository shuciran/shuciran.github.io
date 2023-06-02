### Read user input
```bash
read -p 'Username: ' username
read -sp 'Password: ' password
```

### IF Statement
Take care on spaces between braces and identation:
```bash
if [ <some test> ]
then
  <perform an action>
fi
```

### Useful Scripting Operators
| OPERATOR | DESCRIPTION: EXPRESSION TRUE IF... |
|----------|------------------------------------|
| !EXPRESSION | The EXPRESSION is false. |
| -n STRING | STRING length is greater than zero |
| -z STRING | The length of STRING is zero (empty) |
| STRING1 != STRING2 | STRING1 is not equal to STRING2 |
| STRING1 = STRING2 | STRING1 is equal to STRING2 |
| INTEGER1 -eq INTEGER2 | INTEGER1 is equal to INTEGER2 |
| INTEGER1 -ne INTEGER2 | INTEGER1 is not equal to INTEGER2 |
| INTEGER1 -gt INTEGER2 | INTEGER1 is greater than INTEGER2 |
| INTEGER1 -lt INTEGER2 | INTEGER1 is less than INTEGER2 |
| INTEGER1 -ge INTEGER2 | INTEGER1 is greater than or equal to INTEGER 2 |
| INTEGER1 -le INTEGER2 | INTEGER1 is less than or equal to INTEGER 2 |
| -d FILE | FILE exists and is a directory |
| -e FILE | FILE exists |
| -r FILE | FILE exists and has read permission |
| -s FILE | FILE exists and it is not empty |
| -w FILE | FILE exists and has write permission |
| -x FILE | FILE exists and has execute permission |

### BASH VARIABLES
| VARIABLE NAME | DESCRIPTION |
|---------------|-------------|
| $0 | The name of the Bash script |
| $1 - $9 | The first 9 arguments to the Bash script |
| $# | Number of arguments passed to the Bash script |
| $@ | All arguments passed to the Bash script |
| $? | The exit status of the most recently run process |
| \$$ | The process ID of the current script |
| $USER | The username of the user running the script |
| $HOSTNAME | The hostname of the machine |
| $RANDOM | A random number |
| $LINENO | The current line number in the script |

### Add an Alias
If you need to create a temporary alias just use the command on the terminal, otherwise for persistence create the alias on the .zshrc and run the last command:
```bash
alias mkdir='ping -c 1 localhost'
source ~/.zshrc
```

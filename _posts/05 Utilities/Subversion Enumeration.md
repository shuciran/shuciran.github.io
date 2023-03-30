Port tcp-3690 host a service well-known as Subversion following commands are useful to enumerate such service:
```bash
svn ls svn://10.10.10.203 #list
svn log svn://10.10.10.203 #Commit history
svn checkout svn://10.10.10.203 #Download the repository
svn up -r 2 #Go to revision 2 inside the checkout folder
```
Examples:
[[Worker#^90567c]]

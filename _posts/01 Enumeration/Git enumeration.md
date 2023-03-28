### Githacker
Command to extract the whole git project:
```bash
githacker --url http://10.10.11.134/.git/ --output-folder results
``` 
Examples:
[[Epsilon#^6edabc]]

### Git
Command to list the commits under a git project (you should be under .git folder):
```bash
git log
```
After retrieving the hash for every commit, if we want to see the changes:
```bash
git show <hash-id>
```

Examples:
[[Epsilon#^a27371]]

[GITDUMPER](https://pentester.land/tutorials/2018/10/25/source-code-disclosure-via-exposed-git-folder.html)



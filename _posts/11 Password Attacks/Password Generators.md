### Cewl
The following command scrapes the web site, locates words with a minimum of six characters (-m 6), and writes (-w) the wordlist to a custom file (megacorp-cewl.txt):

```
kali@kali:~$ cewl www.megacorpone.com -m 6 -w megacorp-cewl.txt
```

### John The Ripper

To create passwords that meet this requirement, we could write a Bash script. However, we will instead use a much more powerful tool called John the Ripper (JTR),[2](https://portal.offensive-security.com/courses/pen-200/books-and-videos/modal/modules/password-attacks/wordlists/standard-wordlists#fn2) which is a fast password cracker with several features including the ability to generate custom wordlists and apply rule permutations.

Moving forward with our assumption about the password policy, we will add a rule to the JTR configuration file (/etc/john/john.conf) that will mutate our wordlist, appending two digits to each password. To do this, we must locate the _[List.Rules:Wordlist]_ segment where wordlist mutation rules are defined, and append a new rule. In this example, we will append the two-digit sequence of numbers from (double) zero to ninety-nine after each word in our wordlist.

We will begin this rule with the $ character, which tells John to append a character to the original word in our wordlist. Next, we specify the type of character we want to append, in our case we want any number between zero and nine (_[0-9]_). Finally, to append double-digits, we will simply repeat the _$[0-9]_ sequence. The final rule is shown in Listing 2.

```
kali@kali:~$ sudo nano /etc/john/john.conf
...
# Wordlist mode rules
[List.Rules:Wordlist]
# Try words as they are
:
# Lowercase every pure alphanumeric word
-c >3 !?X l Q
# Capitalize every pure alphanumeric word
-c (?a >2 !?X c Q
# Lowercase and pluralize pure alphabetic words
...
# Try the second half of split passwords
-s x_
-s-c x_ M l Q
# Add two numbers to the end of each password
$[0-9]$[0-9]
...
```

Now that the rule has been added to the configuration file, we can mutate our wordlist, which currently contains 312 entries.

To do this, we will invoke john and specify the dictionary file (--wordlist=megacorp-cewl.txt), activate the rules in the configuration file (--rules), output the results to standard output (--stdout), and redirect that output to a file called mutated.txt:

```
kali@kali:~$ john --wordlist=megacorp-cewl.txt --rules --stdout > mutated.txt
Press 'q' or Ctrl-C to abort, almost any other key for status
46446p  0:00:00:00 100.00% (2018-03-01 15:41) 663514p/s chocolate99

kali@kali:~$ grep Nanobot mutated.txt
...
Nanobot90
Nanobot91
Nanobot92
Nanobot93
Nanobot94
Nanobot95
Nanobot96
...
```

> Listing 3 - Mutating passwords using John the Ripper

### Crunch

| PLACEHOLDER | CHARACTER TRANSLATION |
|-------------|-----------------------|
|@| Lower case alpha characters|
|,|Upper case alpha characters|
|%|Numeric characters|
|^|Special characters including space|

To generate a wordlist that matches our requirements, we will specify a minimum and maximum word length of eight characters (8 8) and describe our rule pattern with -t ,@@^^%%%:

```
kali@kali:~$ crunch 8 8 -t ,@@^^%%%
```
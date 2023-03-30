
NTLM authentication is used when a client authenticates to a server by IP address (instead of by hostname), or if the user attempts to authenticate to a hostname that is not registered on the Active Directory integrated DNS server. Likewise, third-party applications may choose to use NTLM authentication instead of Kerberos authentication.

The NTLM authentication protocol consists of seven steps as shown in Figure 3 and explained in depth below.

![[Pasted image 20221113211410.png]]

In the first authentication step, the computer calculates a cryptographic hash, called the _NTLM hash_, from the user's password. Next, the client computer sends the user name to the server, which returns a random value called the _nonce_ or _challenge_. The client then encrypts the nonce using the NTLM hash, now known as a _response_, and sends it to the server.

The server forwards the response along with the username and the nonce to the domain controller. The validation is then performed by the domain controller, since it already knows the NTLM hash of all users. The domain controller encrypts the challenge itself with the NTLM hash of the supplied username and compares it to the response it received from the server. If the two are equal, the authentication request is successful.

As with any other hash, NTLM cannot be reversed. However, it is considered a "fast-hashing" cryptographic algorithm since short passwords can be cracked in a span of days with even modest equipment.

By using cracking software like Hashcat with top-of-the-line graphic processors, it is possible to test over 600 billion NTLM hashes every second. This means that all eight-character passwords may be tested within 2.5 hours and all nine-character passwords may be tested within 11 days.

Next we will turn to Kerberos, which is the default authentication protocol in Active Directory and for associated services.
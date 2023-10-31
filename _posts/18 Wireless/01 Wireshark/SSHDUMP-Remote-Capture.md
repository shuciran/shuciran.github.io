### SSHDUMP Remote Capture

Let's review how to do remote packet capture with SSH from within Wireshark. If we've previously isolated our devices to only the wireless interfaces, we can refocus to External Capture devices. We'll select External Capture and then de-select all others in the interface's dropdown menu.

![SSHDump](/assets/img/Pasted-image-20230925232200.png)

Next, let's click on the cog wheel to the left of SSH remote capture: sshdump at the bottom of the Capture section to open the options window.

![External-Interfaces](/assets/img/Pasted-image-20230925232238.png)

Wireshark typically captures from interfaces on the local system. These "External Capture" interfaces are using ExtCap,1 which allows executables to be seen as capture interfaces. All of these are separate binaries: ciscodump, dpauxmon, randpkt, sdjournal, sshdump, and udpdump. They provide data in PCAP format and can be found in the /usr/lib/x86_64-linux-gnu/wireshark/extcap/ directory (on a 64bit Kali). Some of these tools have man pages but they all are executed with a few arguments. All of them are similarly configured in the Wireshark GUI.

On this first screen, let's configure the IP address of the remote host. The Remote SSH server port field is required, so we'll enter the default of port 22.

![Configuring-sshdump](/assets/img/Pasted-image-20230925232310.png)

On the Authentication tab, we enter the SSH credentials for the remote host. SSHdump handles both password-based authentication and key-based authentication.

![Authentication-tab](/assets/img/Pasted-image-20230925232350.png)

SSH key-based and ProxyCommand authentication are also options, but we will confine our capture to using a username and password.

In this example, we are authenticating to the remote system as root. To use a standard user instead, you will need to run 'sudo dpkg-reconfigure wireshark-common / yes' to reconfigure the wireshark package and 'sudo usermod -a -G wireshark kali' to add the user (kali in this example) to the wireshark group

Next, we enter the required parameters in the Capture tab.

![SSHDump-capture](/assets/img/Pasted-image-20230925232424.png)

The SSHDump tool is basically an application that builds an sshdump command line argument. The Remote interface textbox is the equivalent of appending -i wlan0mon in the Remote capture command textbox.

The user on the remote system must be able to either initiate a capture or have access to sudo. If it's the latter, check the Use sudo on the remote machine checkbox.

In this example, we will use dumpcap to minimize CPU usage, although we could use any capture tool as long as it is available on the remote capture device.

The last tab is to enable debugging and to specify where to save the log messages in case we encounter errors.

![sshdump-debug](/assets/img/Pasted-image-20230925232515.png)

Once ready, we click Start. If the parameters are correct and the remote system is reachable, the capture will start shortly.

When Save parameter(s) on capture start is checked, the next time SSHdump is used, it won't prompt for settings and will start automatically. If the settings are not properly set and an error results, Wireshark does not make resetting to the defaults easy. They can be reset via Edit > Preferences... > Advanced. In the resulting Search: textbox, we type "sshdump". Then double click every modified parameter (anything in bold) to set SSHDump back to the default values. Click on OK and SSHDump is back to its default configuration.
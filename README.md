# Hardening SSH in Linux

## Configuring the Server Directory Structure

Log into your Debian server via your existing administrative session and set up the .ssh environment for your standard user (never use root's home directory):
```Bash
# Create the .ssh directory with secure permissions
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Create the authorized_keys file with secure permissions
touch ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

Open the file to paste your client's public key with a descriptive identifier/comment at the end:
```Bash
vim ~/.ssh/authorized_keys
```

*Example entry*:
```Plaintext
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... user@laptop personal-laptop
```

## Creating the Hardening Configuration (`hardening.conf`)

Modern Debian uses a modular drop-in directory (`/etc/ssh/sshd_config.d/`) to prevent system upgrades from overwriting your custom rules.

Create and populate your hardening file:
```Bash
sudo vim /etc/ssh/sshd_config.d/hardening.conf
```

Paste the following heavily commented configuration:
```Plaintext
# ==========================================
# SSH Hardening Configuration
# ==========================================

# Disable root login entirely. Use a standard user with sudo.
PermitRootLogin no

# Enforce SSH key-based authentication only. Kill passwords.
PubkeyAuthentication yes
PasswordAuthentication no
PermitEmptyPasswords no

# Limit authentication attempts to slow down brute-forcers
MaxAuthTries 3
```

## Syntax Verification & Service Deployment

Never restart SSH blindly without checking your configuration syntax first.

1. Test the configuration syntax (using absolute path for administrative PATH constraints):
```Bash
sudo /usr/sbin/sshd -t
```

*(If the command returns no output, your configuration syntax is 100% valid).*

2. Verify effective active parameters:
```Bash
sudo /usr/sbin/sshd -T | grep -E "permitrootlogin|passwordauthentication|pubkeyauthentication|maxauthtries"
```

3. Restart the SSH service:
```Bash
sudo systemctl restart ssh
```

## Safety Protocol & Verification

> [!Warning]
> **CRITICAL WARNING**: Do not close your current administrative terminal session until you have verified access from a separate window.

1. Open a brand-new terminal window on your client device.
2. Test the connection:
```Bash
ssh username@your_server_ip
```

3. Confirm you are logged in instantly via your cryptographic key without a password prompt.

---

# Additional Steps

## Modify port

Moving SSH port (`22` by default) into a custom number, such as: `2222` or `8122`. It does not add any real security, however it drastically cleans "noise" and logs from automatic bot attacks from the Internet.

``` text
# Moving SSH port into a custom number.
Port 2222
```

## Set a banner message

This is more related as a legal requirement, but this setting takes a moment. You can provide specific information in banner messages. To write the banner message you need to access into `/etc/issue.net` file. Then, you need to open the `hardening.conf` file and tell it to use the content of `issue.net` as the banner.

**issue.net**
``` text
Warning! Authorized use only.
This server is the property of MyCompanyName.com
```
Tell SSH to use the banner message. Open the `hardening.conf` and find the line that reads **Banner**.

**ssdh_config**
``` text
# Provide specific information in banner messages.
Banner /etc/issue.net
```

---

# SSH Cryptographic Hardening

## Configuring Modern Cryptography

On Debian, modular configuration files inside the drop-in directory keep system overrides clean and isolated from the main configuration.
1. Create a dedicated configuration file:
```Bash
sudo vim /etc/ssh/sshd_config.d/crypto-hardening.conf
```

2. Add the strict cryptographic policies:
```Plaintext
# Modern Key Exchange Algorithms
KexAlgorithms curve25519-sha256,curve25519-sha256@libssh.org,diffie-hellman-group16-sha512,diffie-hellman-group18-sha512

# High-Performance, Secure Ciphers
Ciphers chacha20-poly1305@openssh.com,aes256-gcm@openssh.com,aes128-gcm@openssh.com

# Encrypted-then-MAC (EtM) Message Authentication Codes
MACs hmac-sha2-512-etm@openssh.com,hmac-sha2-256-etm@openssh.com
```

3. Save and close the file

## Syntax Testing and Service Restart

> [!Warning]
> Never restart the SSH daemon without validating syntax first, or you risk locking yourself out.

1. Test the configuration syntax:
```Bash
sudo sshd -t
```

*(If the command returns no output and exits with `0`, the syntax is correct.)*

2. Restart the SSH service to apply the new baseline:
```Bash
sudo systemctl restart ssh
```
# Server Initialization Runbook

Initialize a server for Ansible management.

## Prepare the server

### Install Minimal Debian

1. Boot the server from a Debian netinst USB.
2. Leave the root password blank.
3. Create the `rhew` user with a temporary password.
4. In the Debian installer:
   - Select `SSH server`
   - Select `standard system utilities`
   - Deselect every desktop/task option
5. Complete installation and reboot.

### Secure `rhew` User

1. Add the SSH key from the control machine:

```bash
ssh-copy-id rhew@server-ip
```

2. Log in to the server:

```bash
ssh rhew@server-ip
```

3. Make `rhew` passwordless sudo:

```bash
echo 'rhew ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/rhew
sudo chmod 440 /etc/sudoers.d/rhew
sudo -n true
```

4. Lock password logins:

```bash
sudo passwd -l root
sudo passwd -l rhew
```

5. Return to the control machine and verify:

```bash
exit
ssh rhew@server-ip python3 --version
ssh rhew@server-ip sudo -n true
```

## Expected state

- `rhew` can SSH in with a key and run `sudo` without a password
- The host has `python3` installed
- No local account can log in with a password

Ansible can manage the machine.

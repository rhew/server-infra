# Server Initialization Runbook

Initialize a server for Ansible management.

## Prepare the server

1. Boot the server from a Debian netinst USB.
2. In the Debian installer:
   - Select `SSH server`
   - Select `standard system utilities`
   - Deselect every desktop/task option
3. Create the `rhew` user with a temporary password.
4. Leave the root password blank.
5. Add and verify the SSH key for `rhew`:

```bash
ssh-copy-id rhew@server-ip
ssh rhew@server-ip
```

6. Lock password logins:

```bash
sudo passwd -l root
sudo passwd -l rhew
```

7. Make `rhew` passwordless sudo:

```bash
echo 'rhew ALL=(ALL) NOPASSWD:ALL' | sudo tee /etc/sudoers.d/rhew
sudo chmod 440 /etc/sudoers.d/rhew
```

8. Verify SSH key login and passwordless sudo:

```bash
ssh rhew@server-ip
ssh rhew@server-ip sudo -n true
```

## Expected state

- `rhew` can SSH in with a key and run `sudo` without a password
- The host has `python3` installed
- No local account can log in with a password

Ansible can manage the machine.

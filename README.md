# server-infra

Server initialization runbook and Ansible configuration for my services.

See [server-init-runbook.md](server-init-runbook.md) for the install and bootstrap steps.

See [ansible-guide.md](ansible-guide.md) for what Ansible will manage.

Create local host config:

```bash
cp host_vars/lenny.local.example.yml host_vars/lenny.local.yml
$EDITOR host_vars/lenny.local.yml
```

Pi-hole assigns a random web password on first start.

Run Ansible:

```bash
ansible-playbook --syntax-check playbooks/site.yml
ansible-playbook playbooks/site.yml --list-tasks
ansible-playbook playbooks/site.yml
```

The normal site run starts Pi-hole, the media stack, and the LED wall client.

Pull service repo updates:

```bash
ansible-playbook playbooks/site.yml --tags service_repos
```

This updates the project checkouts. It does not restart containers.

Start individual services:

```bash
ansible-playbook playbooks/site.yml --tags pihole
ansible-playbook playbooks/site.yml --tags media_stack
ansible-playbook playbooks/site.yml --tags led_wall
```

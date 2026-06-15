# server-infra

Server initialization runbook and Ansible configuration for my services.

See [server-init-runbook.md](server-init-runbook.md) for the install and bootstrap steps.

See [ansible-guide.md](ansible-guide.md) for what Ansible will manage.

Create local host config:

```bash
cp host_vars/lenny.local.example.yml host_vars/lenny.local.yml
$EDITOR host_vars/lenny.local.yml
```

Create agent-control-plane config on the server:

```bash
ssh lenny '$EDITOR /home/rhew/agent-control-plane/hermes-config/.env'
```

Use [host_vars/lenny.agent-control-plane.env.example](host_vars/lenny.agent-control-plane.env.example) as the template.

Pi-hole assigns a random web password on first start. Set or change it after startup:

```bash
ssh -tt lenny 'docker exec -it pihole pihole setpassword'
```

Run Ansible:

```bash
ansible-playbook --syntax-check playbooks/site.yml
ansible-playbook playbooks/site.yml --list-tasks
ansible-playbook playbooks/site.yml
```

The normal site run starts Pi-hole, the media stack, and the LED wall client.
It starts `agent-control-plane` after `/home/rhew/agent-control-plane/hermes-config/.env` exists on the server.

Pull service repo updates:

```bash
ansible-playbook playbooks/site.yml --tags service_repos
```

This updates the project checkouts. It does not restart containers.

Check Docker pruning:

```bash
ssh lenny 'systemctl list-timers docker-prune.timer'
```

Start individual services:

```bash
ansible-playbook playbooks/site.yml --tags pihole
ansible-playbook playbooks/site.yml --tags media_stack
ansible-playbook playbooks/site.yml --tags led_wall
ansible-playbook playbooks/site.yml --tags agent_control_plane
```

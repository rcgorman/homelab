# Homelab infrastructure

This repo manages the configuration for my RHEL VMs and their Podman Quadlet services. Ansible deploys the files; application data and generated state stay on the hosts and need separate backups.

## NetBird on edge01

The NetBird stack on `edge01.internal.gormantech.com` is the first service managed here. Its playbook has been run successfully and tested by restoring a changed Quadlet and starting stopped containers.

- `services/netbird/quadlets/` contains the system Quadlets, with image versions pinned in Git.
- `services/netbird/files/` contains configuration that can be copied as is.
- `services/netbird/templates/` contains files rendered with secrets from the encrypted `ansible/inventory/group_vars/all/vault.yml`.
- `ansible/playbooks/deploy-netbird.yml` installs the files, manages the Podman secret, and applies service changes.

Keep the Vault password outside the repository. Live configuration files containing credentials, Traefik logs, and Podman volumes do not belong in Git. The dashboard's `AUTH_CLIENT_SECRET` is empty because I haven't setup keycloak in netbird yet.

## Deploy

From the repository root, install the required collection and review the proposed changes:

```bash
ansible-galaxy collection install -r ansible/requirements.yml
ansible-playbook --syntax-check -i ansible/inventory/hosts.yml ansible/playbooks/deploy-netbird.yml
ansible-playbook --check --diff --vault-id homelab@prompt -i ansible/inventory/hosts.yml ansible/playbooks/deploy-netbird.yml
```

Confirm the named volumes have a backup, then deploy:

```bash
ansible-playbook --vault-id homelab@prompt -i ansible/inventory/hosts.yml ansible/playbooks/deploy-netbird.yml
```

When deploying NetBird over edge01's public IP, disconnect the NetBird client on the controller first. Restarting the stack can interrupt that connection.

Renovate finds Quadlet image updates and creates pull requests. Review and merge a change, pull it locally, then run the playbook manually. There is no automatic deployment yet. I may setup a workflow for that once everything is done.

The server's rendered `config.yaml` currently uses the host's existing `0644` mode even though it contains an encryption key. Its permissions should be tightened after confirming the container can still read it.

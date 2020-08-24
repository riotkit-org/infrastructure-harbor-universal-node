RiotKit Universal Node
======================

Turns a typical cheap virtual node into a dockerized applications environment node. Natively integrates with RiotKit's Harbor 2+ for webservices deployment.

Installs docker, docker-compose, adds users, configures logs rotation, configures security.

First time setup
----------------

```bash
# setup a repository
git clone git@github.com:riotkit-org/infrastructure-harbor-universal-node.git

cd infrastructure-harbor-universal-node
$(./setup-venv.sh)

# set INVENTORY_GIT_URL to a URL of your repository that contains Ansible inventory with hosts.cfg, host_vars/, group_vars/
cp .env-dist .env
nano .env

rkd :setup
rkd :copy-host-defaults
```

Features
--------

- Structure and automation
- User accounts management
- Basic security (firewall ports, fail2ban, partial disk encryption)
- Extensibility
- Native integration with RiotKit Harbor
- Logs rotation and logs privacy management
- Inventory in separate GIT repository

Cheap node specification
------------------------

This set of playbooks needs to run at least on this specification:

- CPU: 1-4 cores
- RAM: 1-4 GB
- HDD: 20-40 GB SSD (including OS)
- OS: Ubuntu 16.04+ or latest Debian Stable or Latest Armbian
- Host type: Physical ARM or KVM

*Notice: The ARM support is not official. We do not have any ARM nodes to test currently, we did in the past.* 

Usage documentation
===================

Operating
---------

Each operation is performed using RKD (RiotKit-Do) using `rkd` command.
It is recommended to work in a virtual env using `$(./setup-venv.sh)` wrapper - for stability and reproducibility.

Design - concept
----------------

The architecture requires that the Ansible Inventory (list of SSH hosts with variables per host to customize deployment roles) should be placed
in a separate repository, and the url should be in `.env` at `INVENTORY_GIT_URL` variable.

Thanks to this concept we can leave this *Universal Node Repository* at github, and all the secrets and personalized settings
have separated in other, private repository - to update *Universal Node Repository* we can just do `git pull`. It can be also forked
into private repository for full customization.

How to create inventory repository?
-----------------------------------

Inventory structure should consist of:

**hosts.cfg (example)**

```bash
[disasterrecovery]
recovery.example.internal

```

**group_vars/all**

This file should be actually created from template by executing command `rkd :copy-host-defaults` first time.
In it you can define deployment settings globally for EACH HOST.

**host_vars/recovery.example.internal.yaml (example name)**

There you should define `ansible_ssh_user`, `ansible_ssh_pass`, `ansible_sudo_pass` and/or private key path.
In this file you can also overwrite deployment settings for this host.

When in `group_vars/all` there is a section for example `default_role_encryption` then you can create a `role_encryption` in `host_vars/recovery.example.internal.yaml`.
The values will be MERGED together, the section would not replace original one but will merge all values recursively.

**ENCRYPTION**

Each host file in host_vars can be encrypted with a different Ansible Vault key, Ansible supports this.
With this combination you can divide access to multiple admins handling administration of different servers.

*TIP: Keep hosts.cfg with minimum of information (only group names and host names) to keep it unencrypted, so you will not have to enter password twice*

**Adding this inventory repository**

```bash
# set url in INVENTORY_GIT_URL
nano .env
```

Editing inventory per host
--------------------------

This command will automatically encrypt existing and new file using AES-256 with Ansible Vault.

```bash
rkd :edit:host-config my-host.org
```

Deploying
---------

```bash
# perform a partial disk encryption (optional)
rkd :encrypt-first-time recovery.example.internal

# provision with users, firewall settings, etc. - a regular deployment
rkd :deploy prepare-machine.yml --host recovery.example.internal --vault

# update operating system with careful
rkd :deploy update-machines.yml --host recovery.example.internal --vault
```

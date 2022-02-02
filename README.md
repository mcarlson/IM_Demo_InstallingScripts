# GovStack Sandbox

This project provides scripts for creating and configuring GovStack sandbox environments.

We use [Ansible](https://www.ansible.com/) for scripting and [Digital Ocean](https://www.digitalocean.com/) for hosting. These Ansible scripts borrow from [ssdnodes-ansible-provision](https://github.com/joelhans/ssdnodes-ansible-provision.git)

See the [Functional demo sandbox tasks](https://docs.google.com/document/d/19TgKog4yiA3Ci6LXUNfK-ui8OdtBZuXniyWYbH4L5LY/edit#heading=h.h9szyt5mczga) for more details.

## Initial setup

1. Ensure you have an Ansible Control Node, e.g. [ansible.egovstack.net](ansible.egovstack.net) on Digital Ocean.
2. Check out a copy of this repo on the control node. We suggest creating unique forks/branches to track changes and regularly committing/pushing them to track configuration changes over time.
3. Copy `vars_with_secret_example.yml` to `vars_with_secret.yml`. It contains shared secrets used across scripts.

## Create Digital Ocean droplets

1. Edit `digital_ocean_token` in `vars_with_secret.yml` to include your API Token and SSH keys for access. To create a Digital Ocean API token, see [API -> Tokens/Keys -> Generate New Token](https://cloud.digitalocean.com/account/api/tokens?i=a99fae&preserveScrollPosition=true). Be sure to create tokens with read/write privileges.
2. Edit `ssh_key_fingerprint` in `vars_with_secret.yml` to include the fingerprint of the Digital Ocean SSH key to be added to new droplets. You can add a new SSH key, or use the existing `host@ansible` SSH key in Digital Ocean. To get the fingerprint of an existing SSH key for an existing Digital Ocean SSH key, see your team's security settings, e.g. [Account Settings -> Security -> SSH Keys](https://cloud.digitalocean.com/account/security?i=a99fae).
3. Edit the hosts file to include your new droplets. Droplets of the same type should have the same prefix, e.g.:

```
...

#eregistration
[ereg]
er1.egovstack.net
er2.egovstack.net

...
```

4. Edit `digitalocean.yml` to include the correct starting image and droplet type, e.g.:

```
    droplet_image:
      er: ubuntu-18-04-x64
    droplet_size:
      er: s-2vcpu-4gb
```

5. Use the `digitalocean.yml` Ansible script to create all droplets:

```
$ ansible-playbook -i ./hosts digitalocean.yml
```

6. Use the `drop.yml` Ansible script to destroy droplets.

```
$ ansible-playbook -i ./hosts drop.yml
```

Be sure to add/remove each droplet's IPs to the Ansible inventory, e.g. by editing `hosts`.

## Set up SSL certificates and SSH access droplets

*TODO: How is this script used?*

1. See [provision.README.md](provision.README.md) for more details on how to configure `provision.yml`.
2. Run the playbook:

```
# ansible-playbook -k provision.yml
```

## Set up Information Mediator

*TODO: Need details on how to set up the existing IM cluster here: ss1-3, cs, etc. This should be a complete step-by-step runbook with troubleshooting and manual configuration steps included.*

1. Edit `password` in `vars_with_secret.yml` to include the plaintext password used to create security server API keys.
2. See `init_ss1.yml` for setting up a security server and `use_certs.yml` for configuring certificates on the central server.
3. Run the appropriate playbooks:

```
# ansible-playbook -k init_ss1.yml
```

## Updating known droplets for SSH access

*TODO: How is this different from the section in provision.yml? When do we need to run it?*

To update the SSH known hosts for all droplets, run:
```
# ansible-playbook -k store_known_hosts.yml
```
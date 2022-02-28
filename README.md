# GovStack Sandbox

This project provides scripts for creating and configuring GovStack sandbox environments.

We use [Ansible](https://www.ansible.com/) for scripting and [Digital Ocean](https://www.digitalocean.com/) for hosting. These Ansible scripts borrow from [ssdnodes-ansible-provision](https://github.com/joelhans/ssdnodes-ansible-provision.git)

See the [Functional demo sandbox tasks](https://docs.google.com/document/d/19TgKog4yiA3Ci6LXUNfK-ui8OdtBZuXniyWYbH4L5LY/edit#heading=h.h9szyt5mczga) for more details on this project.

## Initial setup

1. Ensure you have an Ansible Control Node, e.g. [ansible.egovstack.net](ansible.egovstack.net) on Digital Ocean.
2. Check out a copy of this repo on the control node. We suggest creating unique forks/branches to track changes and regularly committing/pushing them to track configuration changes over time.
3. Copy `vars_with_secret_example.yml` to `vars_with_secret.yml`. It contains shared secrets used across scripts.
4. You may wish to sign up for an ESP that provides SMTP access, e.g. https://app.eu.mailgun.com/. Please set your ESP's SMTP account properties in `vars_with_secret.yml`.


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

NOTE: hostnames must include a number, e.g. er1

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

Note that you may need to run this multiple times to remove all subdomains...

## Set up SSL certificates and SSH access to droplets

1. Add known hosts to all droplets:

```
$ ANSIBLE_HOST_KEY_CHECKING=false ansible-playbook -i hosts store_known_hosts.yml
```

NOTE: if this script fails for any reason, you likely need to clean references to old hosts from the `~/.ssh./known_hosts` file on the Ansible Control Node.

If you see an error like `dig +short er3.egovstack.net` you likely need to wait a minute for the DNS update from `digitalocean.yml` to propagate.

2. This playbook creates non-root user, updates packages, configures SSH access, and generates LetsEncrypt certificates for all droplets. On `ansible.egovstack.net`, hit `<enter>` for the forst `SSH password:` prompt. Use `password` from `vars_with_secret.yml` for the `BECOME password[defaults to SSH password]:` prompt. :

```
$ ansible-playbook -i hosts -k provision.yml --ask-become-pass
```

3. OPTIONAL: Reboot all droplets. On `ansible.egovstack.net`, use `password` from `vars_with_secret.yml` for the `BECOME password:` prompt:

```
$ ansible --ask-become-pass -i hosts -b -m reboot all
```

## Set up Information Mediator

See [Installing X-Road on DigitalOcean](https://docs.google.com/document/d/17B-LnWdMlpIblM7nodchec6uMRCTtyHXM2sJvhN0xcA/edit#) for more details on how to set up the XRoad as an Information Mediator.

## Set up ERegistration

1. Set up eregistration:

```
$ ansible-playbook -i hosts -k ereg_coresystem.yml --ask-become-pass
```

2. Wait several minutes for everything to start up. You can ssh into the host to debug, e.g. from ansible.egovstack.net run `host@ansible:~/wkd/ereg$ ssh root@er3.egovstack.net`. It may be helpful to reboot the host a few times and run `top` or `docker ps` to see which processes are healthy.

3. Ensure keycloak has started up completely, then comment out `KEYCLOAK_USER=$KEYCLOAK_ADMIN_USER` and `KEYCLOAK_PASSWORD=$KEYCLOAK_ADMIN_USER_PASSWORD` in the docker compose, e.g. `# vim /opt/eregistrations/compose/eregistrations/docker-compose.yml` followed by `docker-compose up -d keycloak`

4. Ensure the SMTP settings are correct in keycloak, e.g. https://login.er3.ext.egovstack.net/auth/admin/master/console/#/realms/CH/smtp-settings
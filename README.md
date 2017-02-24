ansible-graylog
===============

Ansible Playbook to provision a Graylog 2.2 server with Nginx and Let's Encrypt.

Tested on Ubuntu 14.04 x64 with a [$5/month DigitalOcean VPS](https://m.do.co/c/5aa134a379d7) (1 vCPU, 512MB of RAM).  The Graylog memory requirements is at least 4GB of RAM, however it's possible to run it on a smaller box given that you give it a big enough swapfile.  The playbook will allocate a 4GB swap by default (you can change the setting in [roles/base/defaults/main.yml](roles/base/defaults/main.yml)).

This playbook will configure a single-node Graylog master server.  The web management is behind an Nginx proxy with SSL enabled using Let's Encrypt.  Let's Encrypt is also configured to automatically renew the SSL cert.

A UFW firewall is enabled by default, allowing only the following ports to be accessed from the outside for added security:

- 22
- 80
- 443

You would want to create separate rules to only allow certain clients to send log messages.  For instance, if you're using GELF UDP, you'd want to create a rule to allow access to port `12201` only from certain IP addresses/networks.


## Components

- Graylog 2.2
- ElasticSearch 2.x
- MongoDB
- Nginx
- Let's Encrypt


## Settings/Variables You Should Change

In [roles/graylog/defaults/main.yml](roles/graylog/defaults/main.yml), change the values of the following vars:

- `graylog_password_secret`

You can use `pwgen` to generate a random password (e.g. `pwgen -N 1 -s 96`)

- `graylog_root_password_sha2`

This is the password for the default `admin` user which you can use to log in to the web interface and configure Graylog. Just pick a password and get its sha2 hash (e.g. `echo -n yourpassword | shasum -a 256`). The default password is `password`.
In [roles/certbot/defaults/main.yml](roles/certbot/defaults/main.yml), change this value:

- `certbot_admin_email`

Let's Encrypt will use this email address to send notifications to the admin of issues in the SSL cert.

In the [production](production) inventory file, entire your own server/host.


## Running the Playbook

`ansible-playbook -i production graylogserver.yml`


## Adding UFW rules

See [https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands] for more examples.

### Allow access to UDP port 12201 from IP address 1.2.3.4

`sudo ufw allow proto udp from 1.2.3.4 to any port 12201`


## Deleting UFW rules

Get the rule number to delete:

`ufw status numbered`

Delete the rule based on the rule number:

`ufw delete rule_number`


## Tips

Once you're logged in to the Graylog web admin, go to **System / Indices -> Indices** settings and make sure you adjust the index rotation and retention strategies so you don't run out of disk space on the server in the future.

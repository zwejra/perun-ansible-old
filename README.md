# Perun Ansible script

This repository serves for easier deployment of Perun server with default configuration.

## Requirements

 - 64-bit Debian system
 - Requires atleast 4GB free disk space
 - Ideally dedicated 2 CPUs and atleast 3GB RAM

## Installation of Ansible

- First you need to install Ansible to your system ([Installation of Ansible](http://docs.ansible.com/ansible/intro_installation.html)), which will be used to install Perun to remote server (or localhost).
- **The minimal version of Ansible is 2.2.0.0!**
- **Your public SSH key must be placed in authorized_keys file in .ssh/ folder of root on the remote server!**
- **SSH must be installed on both sides of communication.**

## Clone this repo

- Now you need to download our Ansible repository from our Github
  - [Perun-Ansible repository](https://github.com/CESNET/perun-ansible)

## Set your passwords in Ansible Vault file

- Passwords and secret keys needed for Perun system are stored in **./group_vars/passwords-default.yml** file in cloned Ansible repository.
- Change password for this file with command: `ansible-vault rekey ./group_vars/passwords-default.yml` (**Default password is set to "test"**!)
- In root of your Ansible repository use command: `ansible-vault edit ./group_vars/passwords-default.yml`.
- Change values of passwords and secret keys to users and services.

## Set Local variables

- Set variables and paths to your certificates in file **./group_vars/perun-servers.yml**
- **Copy your certificates for Apache to Perun server (paths you defined in previous step).**

## Download Oracle DB drivers

- Download Oracle DB drivers 'orai18n’ and 'ojdbc8’ from [here](http://www.oracle.com/technetwork/database/features/jdbc/jdbc-ucp-122-3110062.html) on your system with Ansible (**the system which you will use to run this playbook**).
- You need to register before downloading these files.
- Set paths to these files in **./group_vars/perun-servers.yml** file (variables **ojdbc8_file_path** and **orai18n_file_path**).

## Install necessary certificates

- Put all your certificates you will use for user authentication to path defined by **apache_ca_certificate_path** variable stored in **./group_vars/perun-servers.yml** file.

## Set address of your server in inventories

- In **./inventories/prod** file you must **set hostname or ip address** of your Perun server. If you have some test server, you can set it in ./inventories/test file too.
- Ansible_user variable mentioned behind your hostname is user, which has root privileges (default root).
- For example fill this file with this row: **hostname-perun.com ansible_user=root**

## Run Ansible playbook

- Now you can run ansible script with this command (**you need to be in downloaded Ansible repository**). It uses hostname from ./inventories/prod file. You can change it to ./inventories/test (or dev) to use hostname from another inventory.
  - `ansible-playbook -i inventories/prod --ask-become-pass --ask-vault-pass site.yml`
  - It will ask for your **root password (leave empty if target contains your public ssh key in root authorized_keys) and ansible-vault password** (the file which stores passwords meantioned before)

- Perun should be running after installation on **https://[hostname]/[auth-type]/rpc/**, where [auth-type] is either cert for certificates, non for non-authorized access and ba for basic auth (our initial user perun).
- For example: **https://[hostname]/ba/gui/** will show GUI of Perun. First you need to fill username and password (user: perun, password: the one you set in passwords-default.yml file in password_perun_admin variable).

## After installation

Now you need to do stuff, which is not handled by Ansible script:

- **Generate SSH key for Perun**
  - As perun user use command: `ssh-keygen -t rsa -C "perun@[perun's_address]" -f ~/.ssh/id_rsa`
  - Public part of key must be set in /root/.ssh/authorized_keys on target machines, which have to be accessible for Perun. 

- **Add certificates to truststore**
  - The truststore have to contain certificates of servers, which are contacted by Perun. More precisely, chain of their parent certificates (if they are not distributed with Java / browser). Keytool is used to add them:
  - `keytool -keystore /home/perun/.truststore -importcert -trustcacerts -file /tmp/file_with_cert -alias [cert alias, e.g. rt4.cesnet.cz]`

- **Install slave scripts at slave machines**
  - The slave scripts should be installed at the machines that Perun will control, not at the Perun server!
  - Add APT repository by creating file /etc/apt/sources.list.d/meta_repo.list containing the line `deb ftp://repo.metacentrum.cz/ all main pilot` and run `apt-get update`
  - Install slave scripts for each nedeed service, e.g.: `apt-get install perun-slave-process-passwd` for installation of **passwd service**
  - For **all services** install meta package perun-slave-full: `apt-get install perun-slave-full`

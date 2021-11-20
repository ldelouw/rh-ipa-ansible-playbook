# Ansible Playbooks to set up an IPA Environment

## Author
- Luc de Louw <ldelouw@redhat.com>, <luc@delouw.ch>
- License: GPLv3 or later

## Requirements
- The first master as well as the replicas must be configured with the appropriate upstream DNS servers in /etc/resolv.conf
- All Servers must be set up with the correct FQHN

## Upstream Documentation
Please also see the examples on https://github.com/freeipa/ansible-freeipa

## Usage
In a setup with an external DNS infrastructre, ensure that at least the primary master is white listed to allow nsupdates. This is required to automatically configure the required SRV entries before installing the replicas.

## Preparations
Change the password for the vault file passwd.yml, the default password is "changeme". *ansible-vault rekey passwd.yml*. Edit the passwd.yml file and set some secure passwords for the ipa-admin and Directory Manager users. *ansible-vault edit passwd.yml*

The first two playbooks are installing the IPA Servers and does some basic configs, the later one is setting up the configuration stuff such as user- and hostgroups, hbac, sudoers, trust and so on. 

The playbooks have a number prefix which indicates the sequence in which they should be run.

#### First Master Part I

The first playbook installs the software and creates a certficate signing request (CSR) which can be sent to 
an upstream CA to get signed. The resulting CSR will be fetched to files/<ipa-hostname>-ipa.csr

```
ansible-playbook --ask-vault 01-setup-master.yml
```
### First Master Part II
```
02-setup-master-part2.yml
```
This is the second part of the master setup. Please ensure that the  Subordinate CA-cert is merged together with all upstream 
CA-certs. I.e.

```
cp ca.crt chain.crt
cat subca.crt >> chain.crt
cat ipa-ca.crt >> chain.crt
```

### Replicas
```
ansible-playbook --ask-vault 03-setup-replicas.yml
```

### Config
```
ansible-playbook --ask-vault 04-config.yml
```

### Rename hosts 
This playbook is *NOT* idempotent, do only run it once during implementation/migration phase!

```
ansible-playbook --ask-vault 05-rename-hosts-not-idempotent.yml
```

### Enroll Clients
```
ansible-playbook --ask-vault 06-enroll-clients.yml
```

## Additional playbooks

This are playbooks which are only used in development phase, not in daily usage. 

### Reset 
Un-Enrolls clients, uninstalls replicas and first master to clean up a test environment

```
ansible-playbook ZZ-reset.yml
```

### Un-Enroll clients only
```
ansible-playbook ZZ-reset.yml
```





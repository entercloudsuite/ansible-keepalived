Ansible Role: keepalived 
======================================

[![Build Status](https://travis-ci.org/entercloudsuite/ansible-keepalived.svg?branch=master)](https://travis-ci.org/entercloudsuite/ansible-keepalived)
[![Galaxy](https://img.shields.io/badge/galaxy-entercloudsuite.keepalived-blue.svg?style=flat-square)](https://galaxy.ansible.com/entercloudsuite/keepalived)  

Installs keepalived on Ubuntu 16.04 (Xenial)

## Requirements

This role requires Ansible 2.4 or higher.

## Role Variables

The role defines most of its variables in `defaults/main.yml`:

## Example Playbook

Run with default vars:

```
---

- name: KEEPALIVE_HAPROXY_IP_1_NODE_1
  become: true
  become_user: root
  become_method: sudo
  hosts: server_group_haproxy[0]
  roles:
    - role: entercloudsuite.keepalived
      keepalived_options:
        - name: log-detail
      keepalived_vrrp_scripts:
        chk_haproxy:
          script: '/bin/pidof haproxy'
          weight: 100
          interval: 1

      keepalived_vrrp_instances:
        VI_1:
          interface: ens3
          state: MASTER
          priority: 110
          virtual_router_id: 71

          authentication:
            auth_type: PASS
            auth_pass: 'your_password' 

          virtual_ipaddresses:
            - '10.2.255.1 dev ens3 label ens3:1'

          track_scripts:
            - chk_haproxy

- name: KEEPALIVE_HAPROXY_IP_1_NODE_2
  become: true
  become_user: root
  become_method: sudo
  hosts: server_group_haproxy[1]
  roles:
    - role: entercloudsuite.keepalived
      keepalived_options:
        - name: log-detail
      keepalived_vrrp_scripts:
        chk_haproxy:
          script: '/bin/pidof haproxy'
          weight: 100
          interval: 1

      keepalived_vrrp_instances:
        VI_1:
          interface: ens3
          state: BACKUP
          priority: 100
          virtual_router_id: 71

          authentication:
            auth_type: PASS
            auth_pass: 'your_password'

          virtual_ipaddresses:
            - '10.2.255.1 dev ens3 label ens3:1'

          track_scripts:
            - chk_haproxy
```

## Testing

Tests are performed using [Molecule](http://molecule.readthedocs.org/en/latest/).

Install Molecule or use `docker-compose run --rm molecule` to run a local Docker container, based on the [enterclousuite/molecule](https://hub.docker.com/r/fminzoni/molecule/) project, from where you can use `molecule`.

1. Run `molecule create` to start the target Docker container on your local engine.  
2. Use `molecule login` to log in to the running container.  
3. Edit the role files.  
4. Add other required roles (external) in the molecule/default/requirements.yml file.  
5. Edit the molecule/default/playbook.yml.  
6. Define infra tests under the molecule/default/tests folder using the goos verifier.  
7. When ready, use `molecule converge` to run the Ansible Playbook and `molecule verify` to execute the test suite.  
Note that the converge process starts performing a syntax check of the role.  
Destroy the Docker container with the command `molecule destroy`.   

To run all the steps with just one command, run `molecule test`. 

In order to run the role targeting a VM, use the playbook_deploy.yml file for example with the following command: `ansible-playbook ansible-keepalived/molecule/default/playbook_deploy.yml -i VM_IP_OR_FQDN, -u ubuntu --private-key private.pem`.  

## License

MIT

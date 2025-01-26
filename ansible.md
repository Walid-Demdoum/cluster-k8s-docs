## Execution of an Ansible playbook with command line
### `inventory.yml` contain host which could be an IP @
### `playbook.yml` contain the playbook code
### `username` is the disired user to execute commands from playbook in guest
### `--ask-pass` is used to put `username` ssh password for guest
### `--ask-become-pass` to put sudo password if username is sudoer

```bash
ansible-playbook -i inventory.yml playbook.yml -u username --ask-pass --ask-become-pass
```

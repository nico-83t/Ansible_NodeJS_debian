# Ansible configuration for NodeJS APPs
## Configuration
#### Server side
- Need Debian or Ubuntu 
- Need ssh root access with key pair already configured with client

#### Client side
hosts file
```
# Setup your server(s) ip address in hosts file
[web]
71.85.130.169
```

run
```shell
ansible-playbook -i hosts playbook.yml
```
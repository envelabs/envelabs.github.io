# Ansible

## Control Machine Setup

Add Ansible repository for ubuntu16.04/debian8

```
apt install software-properties-common
apt-add-repository ppa:ansible/ansible
echo "deb http://ppa.launchpad.net/ansible/ansible/ubuntu xenial main" > /etc/apt/sources.list.d/ansible-ansible-jessie.list
apt-get -y update
apt-get install -y curl iputils-ping net-tools less vim --allow-unauthenticated
apt-get install -y python --allow-unauthenticated
apt-get install -y ansible --allow-unauthenticated
```

# Commands exec

check connectiviy

```
ansible -i inventory --private-key=~/.ssh/id_rsa -m ping <target-host>
```

exec ad hoc commands

```
ansible -i inventory --private-key=~/.ssh/id_rsa -a "hostname" <target-host>
```

exec roles

```
ansible-playbook -i inventory --private-key=~/.ssh/id_rsa play.yml -e "host=<target-host>"
```

# Handlers

If a task notifies a handler, then a subsequent task in the same play fails, then the whole playbook run will terminate and the handler will not be called. The workaround is to force the handler exec

```
# playbook with handlers

...
- meta: flush_handlers
...
```

# Ansible execution against AWS

```
AWS_PROFILE=th ansible-playbook -i ec2.py  play.ec2.yml -e "host=<host_var>"
```

# Ansible Vault

Encrypt

```
ansible-vault encrypt <file_to_encrypt>.yml --vault-password-file ~/.<vault_pass>.txt
```

Edit

```
ansible-vault edit <file_to_encrypt>.yml --vault-password-file ~/.<vault_pass>.txt
```

Decrypt

```
ansible-vault decrypt <file_to_encrypt>.yml --vault-password-file ~/.<vault_pass>.txt
```

Varfile format for <file_to_encrypt>.yml

```
---
<name_var_value>: |
  -----BEGIN RSA PRIVATE KEY-----
  MIIEogIBAAKCAQEA8uIMcbA07LJXt3TPpJfAlRaFu8MQYgFXibkafaMRGEBUHFb+
  PKyeJ6ksOzSXKjP1+W7z+593EktxKKoFSUCiHcPl5G/tHhMj8xnp8kf26WHnodc6
  pBEiTBB6czxg9wZE0jZ8DtHkQ7d1++p/8T2lP3x17chpcldwczOB+MjPsqfIW5
  ...
  M0Qmk7uqpYpKN5tFUYgXWKNhhYaGhhUKvJPkguu0Pexl0Px4+x+a42LnCX59JPND
  ehtVWez5f7TasdfivRfkn5tcNqEpTcpQ8pBzLMIuYe/GMtYSWNJXhetLtPMntYyK
  MH4ySw0qqXbqZcZAjZeWIV/UrtXQRlISpdqbqwIDAQABAoIBAG5epFMBRHuO62dV
```

# Ansible provisioning using user_data

```
...
 tasks:
  - name: Launch new instance
    ec2:
      key_name: "{{ ec2_keypair }}"
      user_data: |
               #!/bin/sh
               sudo apt-get install -y python
      volumes:
...
```

```
...
 tasks:
  - name: Launch new instance
    ec2:
      key_name: "{{ ec2_keypair }}"
      group_id: "{{ ec2_security_group }}"
      instance_type: "{{ ec2_instance_type }}"
      image: "{{ ec2_image }}"
      vpc_subnet_id: "{{ ec2_subnet_ids | random }}"
      region: "{{ ec2_region }}"
      instance_tags: '{"Name":"{{ host }}", "id":"{{ host }}"}'
      assign_public_ip: yes
      wait: true
      count: 1
      user_data: "{{ lookup('file', 'base_provisioning.sh') }}"
      volumes:
...
```

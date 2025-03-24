# Lab: Install Ansible 
## Task 1: Install Ansible
1. SSH into the jenkins-server and Install ansible
```
sudo apt install ansible-core
```
2. Check the ansible version
```
ansible --version
```

## Task 2: Automate Infrastructure Provisioning with Ansible
1. Create the Script file to launch 2 more instances.
```
vi ansible_script.yaml
```
2. Copy paste the below code & Save the file using "ESCAPE + :wq!"
```
---
- hosts: localhost
  connection: local

  tasks:
    - name: Execute curl command to get token
      shell: "curl -X PUT 'http://169.254.169.254/latest/api/token' -H 'X-aws-ec2-metadata-token-ttl-seconds: 21600'"
      register: TOKEN

    - name: Get region of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/placement/region/"
      register: region

    - name: Get AMI ID of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/ami-id"
      register: ami_id

    - name: Get keypair of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/public-keys/| cut -c 3-100 "
      register: kp

    - name: Get Instance Type of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' http://169.254.169.254/latest/meta-data/instance-type"
      register: instance_type


    - name: Get subnet id of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/subnet-id"
      register: subnet

    - name: Get security group of instance
      shell: "curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs/$(curl -H 'X-aws-ec2-metadata-token:{{ TOKEN.stdout }}' -v http://169.254.169.254/latest/meta-data/network/interfaces/macs)/security-group-ids/"
      register: secgrp


    - name: Generate SSH keypair
      openssh_keypair:
        force: yes
        path: /home/ubuntu/.ssh/id_rsa

    - name: Get the public key
      shell: cat /home/ubuntu/.ssh/id_rsa.pub
      register: pubkey

    - name: Create EC2 instance
      ec2_instance:
        key_name: "{{ kp.stdout }}"
        security_group: "{{ secgrp.stdout }}"
        instance_type: "{{ instance_type.stdout }}"
        image_id: "{{ ami_id.stdout }}"        
        wait: true
        region: "{{ region.stdout }}"
        tags:
          Name: "{{ item }}"
        vpc_subnet_id: "{{ subnet.stdout }}"
        network:
         assign_public_ip: yes
        user_data: |
           #!/bin/bash
           echo "{{ pubkey.stdout }}" >> /home/ubuntu/.ssh/authorized_keys
      register: ec2var
      loop:
          - managed-node1
          - managed-node2

    - name: Make ansible directory
      file:
        path: /etc/ansible
        state: directory
      become: yes

    - debug:
        msg: "{{ ec2var.results[0].instances[0].private_ip_address }}"

    - debug:
        msg: "{{ ec2var.results[1].instances[0].private_ip_address }}"
```
3. Execute the script
```
ansible-playbook ansible_script.yaml
```
4. Verify the creation of the managed nodes on the AWS Management Console.
5. Once you get the ip addresses, do the following:

```
sudo vi /etc/ansible/hosts
```

Add the prive IP addresses, by pressing "INSERT" 
```
node1 ansible_ssh_host=<node1-private-ip> ansible_ssh_user=ubuntu
node2 ansible_ssh_host=<node2-private-ip> ansible_ssh_user=ubuntu
```
e.g. node1 ansible_ssh_host=172.31.14.113 ansible_ssh_user=ec2-user
     node2 ansible_ssh_host=172.31.2.229 ansible_ssh_user=ec2-user

Save the file using "ESCAPE + :wq!"

list all managed node ip addresses.
```
ansible all --list-hosts
```

### SSH into each of them and set the hostnames.
```
ssh ubuntu@< Replace Node 1 IP >
```
```
sudo hostnamectl set-hostname managed-node-1
```
```
exit
```
```
ssh ubuntu@< Replace Node 2 IP >
```
```
sudo hostnamectl set-hostname managed-node-2
```
```
exit
```

### Use ping module to check if the managed nodes are able to interpret the ansible modules
```
ansible all -m ping
```

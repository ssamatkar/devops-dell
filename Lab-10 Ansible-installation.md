# Lab: Install Ansible 
1. SSH into the jenkins-server and Install ansible
```
sudo pip3 install --upgrade pip
```
```
sudo pip3 install ansible==4.10.0
```
```
pip show ansible
```
Download the script using wget
```
wget https://ansible-script.s3.us-west-1.amazonaws.com/ansible_script.yaml
```

Execute the script
```
ansible-playbook ansible_script.yaml
```

Once you get the ip addresses, do the following:

```
sudo vi /etc/ansible/hosts
```

Add the prive IP addresses, by pressing "INSERT" 
```
node1 ansible_ssh_host=<node1-private-ip> ansible_ssh_user=ec2-user
node2 ansible_ssh_host=<node2-private-ip> ansible_ssh_user=ec2-user
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


# **Automated CI/CD Pipeline for a Containerized Web Application on AWS**  

## **Project Overview**  
This project sets up an automated CI/CD pipeline using:  
- **Terraform** for infrastructure provisioning  
- **Ansible** for configuration management  
- **Jenkins with Maven** for continuous integration  
- **Docker Containers** for continuous deployment  

---

## **Step 1: Infrastructure Setup with Terraform**  

### **1.1 Create a Key Pair**  
Generate a key pair using `ssh-keygen`:  
```bash
ssh-keygen -t rsa 
```

---

### **1.2 Create Terraform Files**  
Create the following Terraform files:  

1. **Provider Configuration** (`provider.tf`)  
```hcl
provider "aws" {
  region = "us-east-1"
}
```

2. **Key Pair** (`key_pair.tf`)  
```hcl
resource "aws_key_pair" "mykeypair" {
  key_name   = var.key_name
  public_key = file(var.public_key)
}
```

3. **EC2 Instance** (`instance.tf`)  
```hcl
resource "aws_instance" "my-machine" {
  ami                    = var.ami_id
  key_name               = var.key_name
  vpc_security_group_ids = [var.sg_id]
  instance_type          = var.ins_type

  tags = {
    Name = "my-instance"
  }
}
```

4. **Variables** (`vars.tf`)  
```hcl
# Change the SG ID. You can use the same SG ID used for your CICD anchor server
# Basically the SG should open ports 22, 80, 8080, 9999, and 4243
variable "sg_id" {
    default = "sg-05129de194fe9156f" # us-east-1
}

# Choose a free tier Ubuntu AMI. You can use below. 
variable "ami_id" {
    default = "ami-0866a3c8686eaeeba" # us-east-1; Ubuntu
}

# We are only using t2.micro for this lab
variable "ins_type" {
    default = "t2.medium"
}

# Replace 'yourname' with your first name
variable "key_name" {
    default = "YourName-CICDlab-KeyPair"
}

variable "public_key" {
    default = "/home/ubuntu/.ssh/id_rsa.pub"   #Ubuntu OS
}
```

---

### **1.3 Apply Terraform Code**  
1. Initialize Terraform:  
```bash
terraform init
```
2. Plan the deployment:  
```bash
terraform plan
```
3. Apply the configuration:  
```bash
terraform apply -auto-approve
```

---

## **Step 2: Configuration with Ansible**  

1. **Update the host-file with the EC2 instance IP address**  
```bash
sudo vi /etc/ansible/hosts
```
2. Paste the Public IP of the EC2 instance and save the file.  

3. **SSH into the Instance**  
```bash
ssh ubuntu@<IP-address>
```

4. **Create an Ansible playbook**  
```bash
vi ansible-script.yaml
```

5. **Paste the following code in the playbook**  
```yaml
---

- name: Start installing Jenkins pre-requisites before installing Jenkins
  hosts: jenkins-server
  become: yes
  become_method: sudo
  gather_facts: no

  tasks:

  - name: Update apt repository with latest packages
    apt:
      update_cache: yes
      upgrade: yes

  - name: Installing jdk17 in Jenkins server
    apt:
      name: openjdk-17-jdk
      update_cache: yes
    become: yes

  - name: Installing jenkins apt repository key
    apt_key:
      url: https://pkg.jenkins.io/debian/jenkins.io-2023.key
      state: present
    become: yes

  - name: Configuring the apt repository
    apt_repository:
      repo: deb https://pkg.jenkins.io/debian binary/
      filename: /etc/apt/sources.list.d/jenkins.list
      state: present
    become: yes

  - name: Update apt-get repository with "apt-get update"
    apt:
      update_cache: yes

  - name: Finally, its time to install Jenkins
    apt: 
      name: jenkins 
      update_cache: yes
    become: yes

  - name: Jenkins is installed. Lets start 'Jenkins' now!
    service: 
      name: jenkins 
      state: started

  - name: Wait until the file /var/lib/jenkins/secrets/initialAdminPassword is present before continuing
    wait_for:
      path: /var/lib/jenkins/secrets/initialAdminPassword

  - name: You can find Jenkins admin password under 'debug'
    command: cat /var/lib/jenkins/secrets/initialAdminPassword
    register: out

  - debug: var=out.stdout_lines
```

6. **Execute the playbook**  
```bash
ansible-playbook ansible-script.yaml
```

---

## **Step 3: Configure Jenkins and Docker**  

### **3.1 Add Jenkins to Docker Group**  
```bash
sudo usermod -aG docker jenkins
```

### **3.2 Update sudoers File**  
Edit sudoers:  
```bash
sudo visudo
```
Add the following line:  
```bash
jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker
```

### **3.3 Restart Jenkins**  
```bash
sudo systemctl restart jenkins
```

---

## **Step 4: Jenkins Configuration**  

1. Get the Initial Password for Jenkins:  
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

2. Open Jenkins in Browser:  
```
http://<Jenkins-Public-IP>:8080/
```

3. Unlock Jenkins and Install Plugins  

4. Add Admin User  

5. Install Maven Plugin  

6. Add Maven in Tool Configuration  
- Name: **Maven**  
- Version: **3.9.5**  

---
## **Step 5: Create a maven project in Jenkins
1. Now you need to make a project for your application build, that selects **New Item** from the Home Page of Jenkins
2. Enter an item name as **hello-world** and select the project as **Maven Project** and then **click OK.**
   ( You will be prompted to the configure page inside the hello-world project.)
3. Go to the "**Source Code Management**" tab, and select Source Code Management as **Git**, Now you need to provide the GitHub Repository **Master Branch URL**
4. Keep all the other values as default and select the "**Build**" Tab and inside Goals and options write "**clean package**"
5. Go to **Post Steps Tab**, select **"Run only if the build succeeds"** then click on **Add post-build** step select **Execute shell** from the drop-down and copy paste the below commands in the shell and **Save**  
```
#!/bin/bash

# Navigate to the Jenkins workspace
cd /home/ubuntu/workspace/hello-world/

# Copy the WAR file to the root directory
cp -f target/hello-world-war-1.0.0.war .

# Stop and remove the existing Docker container (if it exists)
sudo docker stop helloworld-container || true
sudo docker rm helloworld-container || true

# Build the Docker image
sudo docker build -t helloworld-image -f /home/ubuntu/Dockerfile .

# Run the Docker container
sudo docker run -d -p 9999:8080 --name helloworld-container helloworld-image
```
## **Step 5: Build Docker Image using Jenkins**  

1. Create a **Dockerfile**  
```
vi Dockerfile
```
Paste the following code:  
```
# Pull Base Image
FROM tomcat:8-jre8

# Maintainer
MAINTAINER "CloudThat"

# Copy the war file to the images tomcat path
ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/
```

---
* Now you can build your **hello-world project** by Clicking on **"Build Now"**
* Once the build is successful, access the tomcat server page,

To access the Page In Browser Type **"http:// < Your Jenkins Host Public IP >:9999/hello-world-war-1.0.0/"** to see the website
* **Example:** http://3.95.192.77:9999/hello-world-war-1.0.0/
   
## **✅ Success Criteria**  
✔️ Jenkins pipeline successfully builds the Docker image  
✔️ Docker container runs the web application  
✔️ Accessible from the browser on port `9999`  

Example URL:  
```  
http://<Your-Jenkins-Public-IP>:9999/hello-world-war-1.0.0/  
```  

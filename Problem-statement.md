## **Automated CI/CD Pipeline for a Containerized Web Application on AWS**  
 
### This project aims to set up an automated **CI/CD pipeline** using:  

* **Terraform** for infrastructure provisioning  

* **Ansible** for configuration management  

* **Jenkins with Maven** for continuous integration and deployment  

---
 
## **Task 1: Infrastructure Setup with Terraform**  
 
### **1. Provision an Ansible Managed Node**  

- Launch an **Ubuntu EC2 instance (t3.medium)** from the existing **CI/CD EC2 instance** using Terraform.  

- This instance will serve as the **Ansible Managed Node** for configuration and deployment.  
 
### **2. Terraform Configuration**  

- Create a **single Terraform file** or separate files for:  

  - **AWS provider**  

  - **Key Pair** (generated using `ssh-keygen -t rsa`)  

  - **Security Group** (reuse the security group ID of the existing CI/CD machine)  

  - **EC2 instance configuration** using the generated key pair  

  - **Ubuntu AMI-ID** (based on the AWS region)  
 
### **3. Apply Terraform Code**  

- Execute Terraform commands to **provision** the infrastructure and **enable SSH access** from the CI/CD machine to the **Ansible Managed Node**.  

---

![1](https://github.com/user-attachments/assets/b2cdc105-2ead-44b6-b48f-7bd5c701382e)
 
## **Task 2: Configuration with Ansible**  
 
### **1. Use the Following Ansible Playbook to Setup Jenkins & Docker on the Managed Node**  

- Run the **Ansible Playbook** from the **CI/CD machine** which will:  

  ✅ Install **Jenkins**  

  ✅ Install **Docker**  

  ✅ Configure **Jenkins** to start automatically  

  ✅ Retrieve the **Jenkins admin password**  

  ✅ Start and configure **Docker**  
 
#### **Ansible Playbook**:

```yaml

---

- name: Install and Configure Jenkins & Docker

  hosts: all

  become: yes

  become_method: sudo

  gather_facts: no

  tasks:

  - name: Update and upgrade the apt repository

    apt:

      update_cache: yes

      upgrade: yes

  - name: Install OpenJDK 17 (Required for Jenkins)

    apt:

      name: openjdk-17-jdk

      update_cache: yes

  - name: Add Jenkins repository key

    apt_key:

      url: https://pkg.jenkins.io/debian/jenkins.io-2023.key

      state: present

  - name: Add Jenkins repository

    apt_repository:

      repo: deb https://pkg.jenkins.io/debian binary/

      filename: /etc/apt/sources.list.d/jenkins.list

      state: present

  - name: Update apt repository after adding Jenkins repository

    apt:

      update_cache: yes
 
  - name: Install Jenkins

    apt:

      name: jenkins

      update_cache: yes

  - name: Start and enable Jenkins service

    service:

      name: jenkins

      state: started

      enabled: yes

  - name: Wait for Jenkins admin password file to be created

    wait_for:

      path: /var/lib/jenkins/secrets/initialAdminPassword

  - name: Retrieve Jenkins initial admin password

    command: cat /var/lib/jenkins/secrets/initialAdminPassword

    register: jenkins_password

  - debug:

      var: jenkins_password.stdout_lines
 
  - name: Install Docker prerequisite packages

    apt:

      name: ['ca-certificates', 'curl', 'gnupg', 'lsb-release']

      update_cache: yes

      state: latest
 
  - name: Add Docker repository key

    apt_key:

      url: https://download.docker.com/linux/ubuntu/gpg

      state: present

  - name: Add Docker repository

    apt_repository:

      repo: deb https://download.docker.com/linux/ubuntu bionic stable

      state: present

  - name: Install Docker

    apt:

      name: ['docker-ce', 'docker-ce-cli', 'containerd.io']

      update_cache: yes

  - name: Start and enable Docker service

    service:

      name: docker

      state: started

      enabled: yes

  - name: Configure Docker to listen on TCP and Unix sockets

    lineinfile:

      path: /lib/systemd/system/docker.service

      regexp: '^ExecStart='

      line: 'ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:4243 -H unix:///var/run/docker.sock'

  - name: Reload systemd daemon

    command: systemctl daemon-reload

  - name: Restart Docker service

    service:

      name: docker

      state: restarted

```

---
 
## **Task 3: Setup Jenkins and Docker**  

ssh into managed node

switch to root user 

add the fllowing

visudo

jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker
 
### **1. Access Jenkins**  

- Open Jenkins in a web browser using the **public IP** of the managed node.  

- Retrieve the **Jenkins admin password** from:  

  ```

  /var/lib/jenkins/secrets/initialAdminPassword

  ```
 
### **2. Configure Maven in Jenkins**  

- Install **Maven Plugins** in Jenkins.  

- Add **Maven Tool** in Jenkins configuration settings.  
 
### **3. Set Up a Maven Project in Jenkins**  

- Create a **New Maven Project** in Jenkins.  

- Use the following GitHub repository as the source code:  

```
https://github.com/Sruti1512/my-app.git

```  

- Configure Jenkins to:  

  ✅ **Pull the latest code** from GitHub  

  ✅ **Use Maven** to build the application  

  ✅ **Deploy the application using Docker**  

  ✅ **Trigger builds manually** (`Build Now` button)  

Use the below docker file :

# Pull Base Image
```
FROM tomcat:8-jre8
 
# Maintainer

MAINTAINER "CloudThat"
 
# Copy the war file to the images tomcat path

ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/

```
 
### **Final Outcome**  

Once the setup is complete, you will have a **manual CI/CD pipeline** where Jenkins:  

- Fetches the latest code from GitHub  

- Uses Maven to build the application  

- Deploys the application in a Docker container  
 
This pipeline lays the foundation for a fully automated CI/CD workflow! 🚀
<img width="391" alt="image" src="https://github.com/user-attachments/assets/cff932ce-c2fc-4819-bbca-d27965de33cf" />

 

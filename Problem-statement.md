## **Automated CI/CD Pipeline for a Containerized Web Application on AWS**  
![image](https://github.com/user-attachments/assets/de59d3fa-ee43-46b0-9ccf-599ee17ee365)

 
### This project aims to set up an automated **CI/CD pipeline** using:  
* **Terraform** for infrastructure provisioning  
* **Ansible** for configuration management  
* **Jenkins with Maven** for continuous integration
* **Docker Containers** for continous deployment  
---
 
## **Task 1: Infrastructure Setup with Terraform**  
 
### **1. Provision an Ansible Managed Node**  
- Connect to the exisiting CI/CD Machine
- Create a **single Terraform file** or separate files for:  
  - **AWS provider**  
  - **Key Pair** (generated using `ssh-keygen -t rsa`)  
  - **Security Group** (reuse the security group ID of the existing CI/CD machine)  
  - **EC2 instance configuration** using the generated key pair  
  - **Ubuntu AMI-ID** (based on the AWS region)
- The terraform script is expected to create a new instance that will serve as the **Ansible Managed Node**.
 
### **2. Apply Terraform Code**  
- Execute Terraform commands to **provision** the managed node
- Verify the **SSH access** from the CI/CD machine to the **Ansible Managed Node**.  
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
 
### **1. Configure Docker Permissions for Jenkins**  
- **SSH into the Managed Node**  
- Switch to the **root user**:  
  ```
  sudo su
  ```
- Open the sudoers file:  
  ```
  visudo
  ```
- Add the following line at the end:  
  ```
  jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker
  ```
- Save and exit.
 
### **2. Access Jenkins**  
- Open Jenkins in a web browser using the **public IP** of the managed node.  
- Retrieve the **Jenkins admin password** from:  
  ```
  cat /var/lib/jenkins/secrets/initialAdminPassword
  ```
 
### **3. Configure Maven in Jenkins**  
- Install **Maven Plugins** in Jenkins.  
- Add **Maven Tool** in Jenkins configuration settings.  
 
### **4. Set Up a Maven Project in Jenkins**  
- Create a **New Maven Project** in Jenkins.  
- Use the following GitHub repository as the source code:  
```
https://github.com/Sruti1512/my-app.git
```
### **5. Use the Following Dockerfile for Deployment**  
```dockerfile
# Pull Base Image
FROM tomcat:8-jre8
 
# Maintainer
MAINTAINER "CloudThat"
 
# Copy the war file to the image's Tomcat path
ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/
```
- Configure Jenkins to:  
- ✅ **Source Code Management** to use Git 
- ✅ **Add the Goals&Options** in Build to have a clean deployment
- ✅ **Configure the Post Build Steps using execute shell** - Navigate to Jenkins workspace -> Copy the WAR file -> top and Remove Existing Container (if running) -> Build a New Docker Image -> Run the Docker Container 
- ✅ **Trigger build manually** (`Build Now` button)  
---
Once the build is successful, access the container,
To access the Page In Browser Type "http:// < Your Managed node Public IP >:9999/hello-world-war-1.0.0/" to see the website.
 
### **Final Outcome**  
<img width="391" alt="image" src="https://github.com/user-attachments/assets/cff932ce-c2fc-4819-bbca-d27965de33cf" />

# Lab 1: Install Jenkins
 
### Task-0: The first step is to `Manually Launch an EC2 Server` with the below configuration:
 
* **Name:** `CICDLab`
* **AMI Type and OS Version:** `Ubuntu 24.04 LTS`
* **Instance type:** `t2.micro`
* Create a new Keypair with the Name `CICDLab-Keypair`
* In security groups, include ports `22 (SSH)` , `80 (HTTP)` and `8080 (jenkins)`
* Go to Advanced Details -> User Data -> Add the following code which installs Java and Jenkins on the EC2.
```
#!/bin/bash
# Set the hostname
sudo hostnamectl set-hostname CICDLab
 
# Update package lists
sudo apt update
 
# Install required packages
sudo apt install -y wget unzip
 
# Install Java Runtime Environment
sudo apt install -y fontconfig openjdk-17-jre
 
# Check Java version
java -version
 
# Download Jenkins key
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
 
# Add Jenkins to the sources list
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
 
# Update package lists again to include Jenkins
sudo apt-get update
 
# Install Jenkins
sudo apt-get install -y jenkins
 
# Enable and start Jenkins service
sudo systemctl enable jenkins
sudo systemctl start jenkins
 
# Check Jenkins status
sudo systemctl status jenkins
```
![image](https://github.com/user-attachments/assets/cfad541c-55f6-44ca-ad82-3d31a724c10a)
 
* **Configure Storage:** 10 GiB
* Click on `Launch Instance.`
 
### Task-1: Install Java
Once the Anchor EC2 server is up and running, SSH into the machine using `Instance Connect` with the username `ubuntu` and do the following:
Verify Java Installation
```
java -version
```
Verify Jenkins Installation
```
sudo systemctl status jenkins
```
### Task-2: Configure Jenkins Server:
 
Get the **Initial Password** for Jenkins from the below path.
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
   (**Example:** "afbe8d33e25b4b908c0b9f91546f09e6")
 
1. Now, go to the **Web Browser** and enter the Jenkins URL as shown: **http://< Jenkin's Public IP>:8080/**
2. Under Unlock Jenkins, enter the above **initialAdminPassword** & click **Continue**.
3. Click on **Install suggested Plugins** on the Customize Jenkins page.
4. Once the plugins are installed, it gives you the page where you can create a New **Admin User**. 
5. Enter the **User Id** and **Password** followed by **Name and E-Mail ID** then click on **Save & Continue**.
6. In the next step, on the Instance Configuration Page, verify your **Jenkins Public IP** and **Port Number** then click on **Save and Finish**

# Lab: Create a CD Pipeline in Jenkins

### Task-1: Installing Docker 

Connect to your jenkins-server and follow the below Commands to perform lab:
```
sudo su
```
Updating the packages
```
apt update -y
```
Installing the packages
```
apt install curl -y
```
Connecting to url
```
curl -SSL https://get.docker.com/ | sh
```
Checking the status of the docker
```
service docker status
```
If the service is not active, then we need to start the service
```
service docker start
```
To add ubuntu user to docker group, if you are not working as the root user
```
usermod -aG docker ubuntu
```
Checking the version of the docker
```
docker --version
```
```
visudo
```
At the end of the file, add the following line
```
jenkins ALL=(ALL) NOPASSWD: /usr/bin/docker
```
Save and exit the file: Press ESC and **:wq!**

### Task-2: Configure the Jenkins Project to deploy the application

```
cd /home/ubuntu
```
```
vi Dockerfile
```
enter the below:
```
# Pull Base Image
FROM tomcat:8-jre8

# Maintainer
MAINTAINER "CloudThat"

# Copy the war file to the images tomcat path
ADD hello-world-war-1.0.0.war /usr/local/tomcat/webapps/
```

1. Go to your **Jenkins Home page**, click on the **drop-down** on **hello-world project**, and select Configure 
tab.
2. Go to **Post Steps Tab**, select **"Run only if the build succeeds"** then click on **Add post-build** step select **Execute shell** from the drop-down and copy paste the below commands in the shell and **Save**

**Execute shell commands in Jenkins:**
#### Note: You may replace 'yourname' with your actual first name (lines 3 and 5).

```
#!/bin/bash

# Navigate to the Jenkins workspace
cd /home/ubuntu/workspace/hello-world/

# Copy the WAR file to the root directory
cp -f target/hello-world-war-1.0.0.war .

# Stop and remove the existing Docker container (if it exists)
sudo docker stop helloworld-container || true
sudo docker rm helloworld-container || true

# Build the Docker image using the Dockerfile in the /home/ubuntu directory
sudo docker build -t helloworld-image -f /home/ubuntu/Dockerfile .

# Run the Docker container
sudo docker run -d -p 9999:8080 --name helloworld-container helloworld-image

```

* Now you can build your **hello-world project** Either by
1. Clicking on **"Build Now"** or
2. By **making a small change in Github files**.

* Once the build is successful, access the tomcat server page,

To access the Page In Browser Type **"http:// < Your Jenkins Host Public IP >:9999/hello-world-war-1.0.0/"** to see the website
* **Example:** http://3.95.192.77:9999/hello-world-war-1.0.0/

* Note: In the security group add the port number 9999.

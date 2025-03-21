# Lab: Create a CICD pipeline using Jenkinsfile

1. Select New Item from the Home Page of Jenkins
2. Enter an item name as demo-app and select the project as Pipeline Project and then click OK
3. In the pipeline sections paste the following script:
```
pipeline {
    agent any
 
    tools {
        maven 'Maven'  // Ensures Maven is installed
    }
 
    environment {
        DOCKER_IMAGE = 'helloworld-image'
        CONTAINER_NAME = 'helloworld-container'
        DOCKERFILE_PATH = '/home/ubuntu/Dockerfile'
        WORKSPACE_DIR = '/var/lib/jenkins/workspace/demo-app'
        PORT_MAPPING = '9999:8080'
    }
 
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/Sruti1512/my-app.git'
            }
        }
 
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
 
        stage('Copy WAR File') {
            steps {
                sh '''
                cd ${WORKSPACE_DIR}
                cp -f target/hello-world-war-1.0.0.war .
                '''
            }
        }
 
        stage('Build Docker Image') {
            steps {
                sh '''
                sudo docker build -t ${DOCKER_IMAGE} -f ${DOCKERFILE_PATH} ${WORKSPACE_DIR}
                '''
            }
        }
 
        stage('Stop and Remove Existing Container') {
            steps {
                sh '''
                sudo docker stop ${CONTAINER_NAME} || true
                sudo docker rm ${CONTAINER_NAME} || true
                '''
            }
        }
 
        stage('Deploy Docker Container') {
            steps {
                sh '''
                sudo docker run -d -p ${PORT_MAPPING} --name ${CONTAINER_NAME} ${DOCKER_IMAGE}
                '''
            }
        }
    }
 
    post {
        success {
            echo '✅ Deployment Successful!'
        }
        failure {
            echo '❌ Deployment Failed!'
        }
    }
}
```

4. Change the repository URL
5. Save the changes and click on Build now

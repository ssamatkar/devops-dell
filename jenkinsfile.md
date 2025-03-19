# Lab: Create a pipeline using Jenkinsfile

1. Select New Item from the Home Page of Jenkins
2. Enter an item name as demo-app and select the project as Pipeline Project and then click OK
3. In the pipeline sections paste the following script:
```
pipeline {
    agent any
    tools {
        maven 'Maven'  // Ensure Maven is installed and configured
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
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying application...'
                // Add deployment script here (e.g., Docker, Kubernetes, or SCP)
            }
        }
    }
    post {
        success {
            echo 'Build Successful!'
        }
        failure {
            echo 'Build Failed!'
        }
    }
}
```
4. Change the repository URL
5. Save the changes and click on Build now

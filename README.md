pipeline {
    agent any

    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('Git Checkout') {
            steps {
              git branch: 'main', url: 'https://github.com/anthamashok1999/Ekart.git'  
            }
        }
        
        
        stage('Compile') {
            steps {
              sh "mvn clean compile"  
            }
        }
        
        stage('Build') {
            steps {
              sh "mvn clean package -DskipTests=true "  
            }
        }
        stage('Build Docker images') {
            steps {
                script{
                 withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                           sh "docker build -t ekart -f docker/Dockerfile ."
                           sh "docker tag ekart anthamashok1999/ekart:latest"
                    }
                }
            }
        }
      stage('Push Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                           
                           sh "docker push --help ekart anthamashok1999/ekart:latest"
                    }
                }
            }
        }  
        
        stage('Run Docker image') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                           
                           sh "docker run --help -d --name ekart -p 8070:8070  anthamashok1999/ekart:latest"
                    }
                }
            }
        }  
    }
}

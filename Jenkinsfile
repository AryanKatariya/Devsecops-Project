pipeline {
    agent any
    tools {
        jdk 'JAVA_11'
        maven 'Maven_3.9.8'
    }

    stages {
        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github_token', url: 'https://github.com/AryanKatariya/Webgoat-Devsecops.git'
            }
        }
        stage('Build application') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }
    }    
}


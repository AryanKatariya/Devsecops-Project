pipeline {
    agent any

    tools {
        jdk 'JAVA_11'
        maven 'Maven_3.9.8'
    }

    environment {
        GITHUB_TOKEN = credentials('github_token')
    }

    stages {
        stage('Cleanup workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github_token', url: 'https://github.com/AryanKatariya/Devsecops-Project.git'
            }
        }
        
        stage('Check secrets') {
            steps {
                sh 'trufflehog https://$GITHUB_TOKEN@github.com/AryanKatariya/Devsecops-Project.git --json > trufflehog_output.json || true'
            }
        }

        stage('SCA - OWASPDC') {
            steps {
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'OWASP-DC'

                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }

        stage ('SAST - SonarQube') {
            steps {
                withSonarQubeEnv('sonar') {
                sh 'mvn clean sonar:sonar -Dsonar.java.binaries=src'
                }
            }
        }

        stage('Build application') {
            steps {
                sh "mvn clean install -DskipTests"
            }
        }

        stage('Deploy to server') {
            steps {
                script {
                    def username = 'dockeradmin'
                    def password = 'docker'
                    def CONTAINER_NAME = 'devops-container'
                    
                    def warFile = '/var/lib/jenkins/workspace/GoatPipeline@2/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar'
                    sh "sshpass -p ${password} scp -o StrictHostKeyChecking=no ${warFile} ${username}@172.31.44.98:/home/dockeradmin"

                    sh """
                            sshpass -p 'docker' ssh -o StrictHostKeyChecking=no dockeradmin@172.31.44.98 \
                            "/usr/sbin/fuser -k 8080/tcp || true && nohup java -jar webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 &"


                        """
                }
            }
        }
    }    
}

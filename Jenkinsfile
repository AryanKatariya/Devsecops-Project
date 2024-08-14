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

//        stage('Start-SonarQube') {
//            steps {
//                script {
//                    sh 'sudo docker stop sonar || true'
//                    sh 'sudo docker rm sonar || true'
//                    sh 'sudo docker run -d --name sonar -p 9000:9000 -v sonarqube_data:/opt/sonarqube/data sonarqube:lts-community'
//                }
//            }
//        }

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
                timeout(time: 3, unit: 'MINUTES'){
                    script {
                        def warFile = '/var/lib/jenkins/workspace/GoatPipeline@2/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar'
                        
                        sshagent(['deploy-ssh']) {
                            sh """
                                scp -o StrictHostKeyChecking=no ${warFile} ubuntu@172.31.8.167:/WebGoat
                            """
                            
                            sh """
                                ssh -o StrictHostKeyChecking=no ubuntu@172.31.8.167 \
                                "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar --server.address=0.0.0.0 > logfile.log 2>&1 & disown"
                            """
                        }
                    }
                }
            }
        }


        stage ('DAST - OWASP ZAP') {
            steps {
                sshagent(['zap-ssh']) {
                    sh """
                    ssh -o StrictHostKeyChecking=no zap@172.31.12.108 \
                    'echo "zap" | sudo docker run --rm -v /home/zap:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://3.108.238.155:8080/WebGoat -x zap_report || true'
                    """
                }
            }
        }
    }    
}
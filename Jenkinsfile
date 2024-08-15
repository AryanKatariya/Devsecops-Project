pipeline {
    agent any

    tools {
        jdk 'JAVA_11'
        maven 'Maven_3.9.8'
    }

    environment {
        GITHUB_TOKEN = credentials('github_token')
        API_KEY = credentials('dojo-api')
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
                sh '''
                    curl -X POST "http://15.206.72.41:8080/api/v2/import-scan/" \
                        -H "Authorization: Token ${API_KEY}" \
                        -F "file=@trufflehog_output.json" \
                        -F "scan_type=Trufflehog Scan" \
                        -F "engagement=2" \
                        -F "version=1.0"
                    '''
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

                sh '''
                    curl -X POST "http://15.206.72.41:8080/api/v2/import-scan/" \
                    -H "Authorization: Token ${API_KEY}" \
                    -F "file=@dependency-check-report.xml" \
                    -F "scan_type=Dependency Check Scan" \
                    -F "engagement=1" \
                    -F "version=1.0"
                '''
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


        stage ('DAST - OWASP ZAP') {
            steps {
                sshagent(['deploy-ssh']) {
                    sh '''
                        sleep 30
                        
                        ssh -o StrictHostKeyChecking=no ubuntu@172.31.12.108 \
                        'echo "ubuntu" | sudo docker run --rm -v /home/ubuntu:/zap/wrk/:rw -t zaproxy/zap-stable zap-full-scan.py -t http://13.200.243.90:8080/WebGoat -x zap_report.yml' || true 
                    
                        ssh -o StrictHostKeyChecking=no ubuntu@172.31.12.108 \
                        "curl -X POST 'http://15.206.72.41:8080/api/v2/import-scan/' \
                        -H 'Authorization: Token ${API_KEY}' \
                        -F 'scan_type=ZAP Scan' \
                        -F 'file=@/home/ubuntu/zap_report.yml' \
                        -F 'engagement=3' \
                        -F 'version=1.0'"
                    '''
                }
            }
        }
    }    
}
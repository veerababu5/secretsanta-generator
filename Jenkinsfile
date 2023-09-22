pipeline {
    agent any
    tools{
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME= tool 'sonar-scanner'
    }

    stages {
        stage('Git checkout') {
            steps {
                git branch: 'master', changelog: false, credentialsId: 'git-cred', poll: false, url: 'https://github.com/veerababu5/secretsanta-generator.git'
            }
        }
        
        stage('COMPILE') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('OWASP Scan') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Sonarqube') {
            steps {
                withSonarQubeEnv('sonar-server'){
                   sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=secretsanta \
                   -Dsonar.java.binaries=. \
                   -Dsonar.projectKey=secretsanta '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build & Push') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker build -t secretsanta ."
                        sh "docker tag  secretsanta veeru3488/secretsanta:latest"
                        sh "docker push veeru3488/secretsanta:latest"
                    }
                }
            }
        }
        
        stage('Docker Deploy') {
            steps {
                script{
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        
                        sh "docker run -d --name santa -p 8080:8080 veeru3488/secretsanta:latest"
                    }
                }
            }
        }
        
        
    }
}

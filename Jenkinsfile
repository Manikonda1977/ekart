pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'https://github.com/lokesh2123/ekart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
         stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=ekart \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=ekart '''
                }
            }
        }

        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        
        stage('OWASP Dependency Check') {
            steps {
                dependencyCheck additionalArguments: '--scan target/', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '#Credentials for docker ', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart lokesh2123/ekart:latest"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '#Credentials for docker', toolName: 'docker') {
                        sh "docker push lokesh/ekart:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '#Credentials for docker', toolName: 'docker') {
                        sh "docker run -d --name shopping -p 8070:8070 adijaiswal/ekart:latest"
                    }
                }
            }
        }
    }
}

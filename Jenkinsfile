pipeline {
    agent any
    
    environment {
        SCANNER_HOME= tool 'sonar-scanner'
    }
    tools {
        maven "maven3"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', credentialsId: 'git-cred', url: 'https://github.com/sachi3050/Boardgame.git'
            }
        }   
        stage('Compile') {
            steps {
            sh 'mvn compile -DskipTests'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test -DskipTests'
            }
        }
        stage('File System Scann') {
            steps {
                sh "trivy fs --format table -o fs.html . "
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }
    
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectKey=Aurora-AI -Dsonar.projectName=Aurora-AI \
                    -Dsonar.java.binaries=.
                    '''
                }
            }
        }
        stage('Artifact Builds and Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'Aurora-AI-Config_File', maven: 'maven3', traceability: true) {
                    sh "mvn deploy -DskipTests"
                }
            }
        }
        stage('Docker build') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t sachidananda06/aurora-ai:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image.html sachidananda06/aurora-ai:latest"
            }
        }
        
        stage('Docker Push Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push sachidananda06/aurora-ai:latest"
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'sachi-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2A0D801E733340D28A29D3F2B2F22B7E.gr7.us-east-1.eks.amazonaws.com') {
                        sh "kubectl apply -f deployment-service.yaml -n webapps"  // Ensure you have the MySQL deployment YAML ready
                    }
                }
            }
        }
        stage('Verify Deployment') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: 'sachi-cluster', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://2A0D801E733340D28A29D3F2B2F22B7E.gr7.us-east-1.eks.amazonaws.com') {
                        sh """
                        kubectl get pods -n webapps
                        kubectl get svc -n webapps
                        """
                    }
                }
            }
        }
    }
}

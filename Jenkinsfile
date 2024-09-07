pipeline {
    agent any
    
    tools {
        maven "maven3"
    }
    
    environment {
        SCANNER_HOME = tool "sonar-scanner"
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Itzmesahil/Owner-avatar-Multi-Tier-With-Database-.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs-report.html --scanners vuln --skip-db-update ."
            }
        }
        
        stage('Publish to Nexus') {
            steps {
                withMaven(globalMavenSettingsConfig: 'settings-maven', maven: 'maven3', traceability: true) {
                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
        
        stage('Docker Build Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker build -t itzmesahil/bankapp:latest ."
                    }
                }
            }
        }
        
        stage('Trivy Image Scan') {
            steps {
                sh "trivy image --format table -o image-scan-report.html itzmesahil/bankapp:latest"
            }
        }
        
        stage('Docker Push To Hub') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred') {
                        sh "docker push itzmesahil/bankapp:latest"
                    }
                }
            }
        }
        
        stage('Deploy To K8s') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS_CLOUD', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://0187BF39CDA429E25E51A44E1E590E23.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl apply -f ds.yaml -n webapps"
                    sleep 30
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                withKubeCredentials(kubectlCredentials: [[caCertificate: '', clusterName: 'EKS_CLOUD', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', serverUrl: 'https://0187BF39CDA429E25E51A44E1E590E23.gr7.ap-south-1.eks.amazonaws.com']]) {
                    sh "kubectl get po -n webapps"
                    sh "kubectl get svc -n webapps"
                }
            }
        }
    }
}

pipeline {
    agent any
    environment {
        DOCKER_USER = "budisarah" 
        GIT_REPO_URL = "https://github.com/Shinbazakura/tokobuku-app.git"
    }
    stages {
        stage('Checkout Code') {
            steps {
                // Mengambil kode dari repository sesuai basis jenkins-pipeline.txt
                git branch: 'master', url: "${GIT_REPO_URL}"
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-login', passwordVariable: 'PASS', usernameVariable: 'USER')]) {
                        // Build images
                        sh "docker build -t ${USER}/tokobuku-backend:latest ./backend"
                        sh "docker build -t ${USER}/tokobuku-frontend:latest ./frontend"
                        
                        // Login dan Push
                        sh "echo ${PASS} | docker login -u ${USER} --password-stdin"
                        sh "docker push ${USER}/tokobuku-backend:latest"
                        sh "docker push ${USER}/tokobuku-frontend:latest"
                    }
                }
            }
        }
        stage('Deploy ke Azure AKS') {
            steps {
                // Menggunakan plugin withKubeConfig sesuai basis jenkins-pipeline.txt
                withKubeConfig([credentialsId: 'aks-config']) {
                    // Apply konfigurasi K8s (Backend & Frontend)
                    sh "kubectl apply -f tokobuku-k8s.yaml"
                    
                    // Apply konfigurasi Ingress (Penting untuk akses via External IP)
                    sh "kubectl apply -f tokobuku-ingress.yaml"
                    
                    // Restart deployment untuk memastikan image terbaru ditarik
                    sh "kubectl rollout restart deployment backend-tokobuku"
                    sh "kubectl rollout restart deployment frontend-tokobuku"
                }
            }
        }
    }
}

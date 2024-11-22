pipeline {
    agent any

    environment {
        SONAR_HOST_URL = 'http://172.28.95.37:9000'
        PROJECT_KEY = 'jobportal'
        SOURCE_DIR = 'Job_portal_CI_CD/jobportal-application'  // Adjust this path if needed
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    def gitRepoUrl = 'https://github.com/agraharibrijesh/devops_pract.git'
                    checkout([$class: 'GitSCM', branches: [[name: 'main']],
                        userRemoteConfigs: [[url: gitRepoUrl]]])
                }
            }
        }

        stage('Static code analysis: Sonarqube') {
            steps {
                def scannerHome = tool 'SonarQube'
                script {
                    // Define the SonarQube credentials ID
                    //def sonarQubeCredentialsId = 'sonar_token1' // Jenkins secret ID for SonarQube token
                    
                    // Run SonarQube analysis using sonar-scanner
                    withSonarQubeEnv('sonar_token1') {
                        sh """/var/lib/jenkins/tools/hudson.plugins.sonar.SonarRunnerInstallation/SonarQube/bin/sonar-scanner \
                        -D sonar.projectVersion=1.0-SNAPSHOT \
                        -D sonar.login=admin \
                        -D sonar.password=Admin@123456 \
                        -D sonar.projectBaseDir=/var/lib/jenkins/workspace/Job_portal_CI_CD/ \
                            -D sonar.projectKey=jobportal \
                            -D sonar.sourceEncoding=UTF-8 \
                            -D sonar.language=python \
                            -D sonar.sources=Job_portal_CI_CD/jobportal-application/accounts \
                            -D sonar.tests=Job_portal_CI_CD/jobportal-application/accounts/migrations \
                            -D sonar.host.url=http://172.28.95.37:9000/"""
                    }
                }
            }
        }

        stage('Quality Gate Status Check : sonar_token1') {
            steps {
                script {
                    // Poll for SonarQube Quality Gate status
                    def qualityGate = waitForQualityGate()
                    
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    } else {
                        echo "Quality Gate passed: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('Build-image and push-docker hub') {
            steps {
                script {
                    // Clean up old Docker images
                    sh "docker images -q | grep -v '67a4b1138d2d' | xargs -I {} docker rmi -f {}"
                    
                    // Build Docker images using docker-compose
                    sh "sudo docker-compose build"
                    
                    // Login to Docker Hub and push the image
                    withCredentials([usernamePassword(credentialsId: 'dockercred', 
                        usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "sudo docker push brijesh35/my-jobportal-app:jp17"
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                script {
                    // Deploy to Kubernetes (assuming Minikube)
                    sh "kubectl config use-context minikube"
                    sh "kubectl apply -f job_dep.yml"
                    sh "kubectl apply -f service.yml"
                    sh "kubectl get pod"
                    sh "kubectl get deployment"
                }
            }
        }
    }
}

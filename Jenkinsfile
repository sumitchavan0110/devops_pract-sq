pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http://172.28.95.37:9000'
        PROJECT_KEY = 'jobportal'
        //SONARQUBE_TOKEN = credentials('sonarv1') 
        SONARQUBE_TOKEN = 'sqp_d71cf354e237a65f6011cd03657d2fa738614901'
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

        stage('Run SonarQube Analysis') {
            steps {
                script {
                    // Run SonarQube analysis using SonarQube scanner
        
                        sh 'sudo sonar-scanner -X'
                
                
                }
            }
        }
        


        /* stage('Quality Gate Status Check') {
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
        } */

        stage('Build-image and push-docker hub') {
            steps {
                script {
                    // Clean up old Docker images
                  //  sh "docker images -q | grep -v '67a4b1138d2d' | xargs -I {} docker rmi -f {}"
                    sh "docker images -q | grep -v '67a4b1138d2d' | grep -v 'e3295fe6246a' | xargs -I {} docker rmi -f {}"
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

                   withCredentials([kubeConfig(credentialsId: 'mymini_cred')]) {
                    sh "kubectl apply -f job_dep.yml"
                    sh "kubectl apply -f service.yml"
                    sh "kubectl get pod"
                    sh "kubectl get deployment"
                    
                   }
      
                    // Deploy to Kubernetes (assuming Minikube)
                    //sh "kubectl config use-context minikube"
                  
                }
            }
        }
    }
}

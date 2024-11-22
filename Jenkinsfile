pipeline {
    agent any

        environment {
        SONAR_HOST_URL = 'http://172.28.95.37:9000'
        SONAR_LOGIN = 'sonar_token1'  // Use your actual SonarQube token
        PROJECT_KEY = 'jobportal'
        SOURCE_DIR = './src'  // Adjust this path if needed
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
                script {
                    // Define the SonarQube credentials ID
                    def SonarQubecredentialsId = 'sonar_token1'
                    // Run SonarQube analysis using sonar-scanner (or equivalent Python tool)
                    withSonarQubeEnv(credentialsId: SonarQubecredentialsId) {
                   sh """
                        sonar-scanner \
                            -Dsonar.projectKey=${PROJECT_KEY} \
                            -Dsonar.sources=${SOURCE_DIR} \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=${SONAR_LOGIN}
                    """
                    }
                }
            }
        }
       stage('Quality Gate Status Check : sonar_token1'){
            steps{
               script{
                    // Define the SonarQube credentials ID
                    def SonarQubecredentialsId = 'sonar_token1'

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
                        //sh 'sudo docker rmi -f \$(docker images -q)'
                        sh "docker images -q | grep -v '67a4b1138d2d' | xargs -I {} docker rmi -f {}"
                        sh "sudo docker-compose build"
                           withCredentials([usernamePassword(credentialsId: 'dockercred', 
                           usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "sudo docker push brijesh35/my-jobportal-app:jp17"
                    }
                }
            }
        } 
        
        stage('kubernetes deployment') {
            steps {
                script {
                        sh "kubectl config use-context minikube"
                        sh "kubectl apply -f job_dep.yml"                            
                        sh "kubectl apply -f service.yml"
                        sh  "kubectl get pod"
                        sh  "kubectl get deployment"                    
                }
            }
        }
    }
}

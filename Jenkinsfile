pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    def gitRepoUrl = 'https://github.com/agraharibrijesh/devops_pract.git'
                
                    checkout([$class: 'GitSCM', branches: [[name: 'main']], userRemoteConfigs: [[url: gitRepoUrl]]])
                }
            }
        }
        
        
        stage('docker-compose build') {
            steps {
                script {
                        //sh 'sudo docker rmi -f \$(docker images -q)'
                        sh "docker images -q | grep -v '67a4b1138d2d' | xargs -I {} docker rmi -f {}"
                        sh "sudo docker-compose build"
                           withCredentials([usernamePassword(credentialsId: 'dockercred', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    }
                    
                    
                        
                          
                    // Push the Docker image to Docker Hub
                    sh "sudo docker push brijesh35/my-jobportal-app:latest"
                    
                    

                }
                
                
            }
        } 
        
        stage('kubernetes') {
            steps {
                script {
                   withKubeConfig([credentialsId: 'mymini_cred']) {
                   
                   
                        
                        //sh "kubectl delete deployment django-app-1"

                            try {
                                // Check if the deployment exists
                                sh 'kubectl get deployment django-app-1'
                                sh 'kubectl get service django-app-1'
                                
                                // If the deployment exists, delete it
                                sh 'kubectl delete deployment django-app-1'
                                sh 'kubectl delete service django-app-1'

                            } catch (Exception e) {
                                // If the deployment doesn't exist, skip the deletion
                                echo 'Deployment django-app-1 does not exist. Skipping deletion.'
                            }

                        sh "kubectl apply -f job_dep.yml"
                        
                        sh "kubectl expose deployment django-app-1 --type=NodePort --port=8000"
                        sh  "kubectl get pod"
                        sh  "kubectl get deployment"
                                         
                   }
                }
            }
        }
    }
}

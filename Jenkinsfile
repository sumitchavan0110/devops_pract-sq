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
        
        
        /* stage('docker-compose build') {
            steps {
                script {
                        
                        //sh "sudo docker-compose build"
                           withCredentials([usernamePassword(credentialsId: 'dockercred', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                    }
                    
                    
                        
                          
                    // Push the Docker image to Docker Hub
                    //sh "sudo docker push brijesh35/my-jobportal-app:latest"
                    
                    

                }
                
                
            }
        } */
        
        stage('kubernetes') {
            steps {
                script {
                   withKubeConfig([credentialsId: 'mymini_cred']) {
                   
                   
                       
                       // sh "minikube profile list"  
                       
                         //sh "/usr/local/bin/minikube delete"
                        //sh "/usr/local/bin/minikube start"
                         //sh "kubectl apply -f job_dep.yml"
                         //sh "kubectl expose deployment django-app-1 --type=NodePort --port=8000"
                         //sh "/usr/local/bi/minikube service django-app-1"
                         //sh "/usr/local/bin/minikube service --all"
                          //sh "minikube service django-app-1 --profile=minikube --url"
                     
                     
                   
                      
                     
                      //sh "minikube dashboard --url"
                      
                      sh  "kubectl get pod"
                      sh  "kubectl get deployment"
                      
                      //sh  "kubectl get service"
                      
                      
                  
                    
                       
                   }
                }
            }
        }
    }
}

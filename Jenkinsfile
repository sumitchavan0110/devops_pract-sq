pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http:localhost:9000'
        PROJECT_KEY = 'jobportal'
        //SONARQUBE_TOKEN = credentials('sonarv1') 
        SONARQUBE_TOKEN = 'squ_a83d99b633295383cf047924a5005967029dadbe'
        SOURCE_DIR = 'Job_portal_CI_CD/jobportal-application'  // Adjust this path if needed

        DOCKER_IMAGE = 'sumitchavan0110/jobportalimage:v1'
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    def gitRepoUrl = 'https://github.com/sumitchavan0110/devops_pract-sq.git'
                    checkout([$class: 'GitSCM', branches: [[name: 'main']],
                        userRemoteConfigs: [[url: gitRepoUrl]]])
                }
            }
        }

         stage('Run SonarQube Analysis') {
            steps {
                
                script {
                    // Run SonarQube analysis using SonarQube scanner
                       withSonarQubeEnv('MySonarQube') {
                        sh 'sudo sonar-scanner -X'
                        }
                    }

                 /*timeout(time: 10, unit: 'MINUTES') {
                 script {
                            waitForQualityGate abortPipeline: true
                        }
                   } */
                }

            }
        

         stage('Quality Gate Status Check') {
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




        

        stage('Build-image') {
            steps {
                script {
                    // Clean up old Docker images
                  //  sh "docker images -q | grep -v '67a4b1138d2d' | xargs -I {} docker rmi -f {}"
                    sh "docker images -q | grep -v '67a4b1138d2d' | grep -v 'e3295fe6246a' | xargs -I {} docker rmi -f {}"
                    // Build Docker images using docker-compose
                    sh "sudo docker-compose build"
                    
                    // Login to Docker Hub and push the image
                    /* withCredentials([usernamePassword(credentialsId: 'dockercred', 
                        usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh "sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD"
                        sh "sudo docker push brijesh35/my-jobportal-app:jp17" */
                    }
                }
            }
        

stage('Run Trivy Scan') {
    steps {
        script {
            def result = sh(
                // Run Trivy to scan the Docker image for vulnerabilities
                script: "trivy image --exit-code 1 --no-progress --format table -o trivy-report.txt ${DOCKER_IMAGE}",
                returnStatus: true
            )
            echo "Trivy scan completed with exit code: ${result}"
            if (result != 0) {
                echo "Vulnerabilities found in the image. Proceeding with the build."
            }

            // List files in the workspace for debugging
            sh "ls -l"
        }
    }
}

stage('Publish Trivy Report') {
    steps {
        script {
            // Verify if the file exists before archiving
            sh "ls -l trivy-report.txt"

            // Archive the Trivy scan report in table format as an artifact in Jenkins
            archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
        }
    }
}




       /* stage('Fail on Critical Vulnerabilities') {
            steps {
                script {
                    // Check the Trivy report for critical vulnerabilities
                    def report = readJSON file: 'trivy-report.json'
                    def criticalVulnerabilities = report.findAll { it.severity == 'CRITICAL' }
                    
                    if (criticalVulnerabilities.size() > 0) {
                        error "Critical vulnerabilities found in the Docker image!"
                    }
                }
            }
        } */

        stage('push image-docker hub') {
            steps {
                script {
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

                   withKubeConfig([credentialsId: 'mymini_cred']) {
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

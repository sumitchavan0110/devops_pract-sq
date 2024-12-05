pipeline {
    agent any

    environment {
        SONARQUBE_SERVER = 'http://localhost:9000'
        PROJECT_KEY = 'jobportal'
        SONARQUBE_TOKEN = 'squ_a83d99b633295383cf047924a5005967029dadbe'
        SOURCE_DIR = 'Job_portal_CI_CD/jobportal-application'
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
                    withSonarQubeEnv('MySonarQube') {
                        sh '''
                            sonar-scanner \
                            -Dsonar.projectKey=${PROJECT_KEY} \
                            -Dsonar.sources=${SOURCE_DIR} \
                            -Dsonar.host.url=${SONARQUBE_SERVER} \
                            -Dsonar.login=${SONARQUBE_TOKEN}
                        '''
                    }
                }
            }
        }

        stage('Quality Gate Status Check') {
            steps {
                script {
                    def qualityGate = waitForQualityGate()
                    if (qualityGate.status != 'OK') {
                        error "Quality Gate failed: ${qualityGate.status}"
                    } else {
                        echo "Quality Gate passed: ${qualityGate.status}"
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh '''
                        docker images -q | grep -v '67a4b1138d2d' | grep -v 'e3295fe6246a' | xargs -I {} docker rmi -f {}
                        docker-compose build
                    '''
                }
            }
        }

        /*
        stage('Run Trivy Scan') {
            steps {
                script {
                    def result = sh(
                        script: "trivy image --exit-code 1 --no-progress --format table -o trivy-report.txt ${DOCKER_IMAGE}",
                        returnStatus: true
                    )
                    echo "Trivy scan completed with exit code: ${result}"
                    if (result != 0) {
                        echo "Vulnerabilities found in the image. Please review the Trivy report."
                    }
                }
            }
        }

        stage('Publish Trivy Report') {
            steps {
                script {
                    sh "ls -l trivy-report.txt"
                    archiveArtifacts artifacts: 'trivy-report.txt', allowEmptyArchive: true
                }
            }
        }
        */

        stage('Push Image to Docker Hub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'dockercred', 
                        usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_PASSWORD')]) {
                        sh '''
                            docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD
                            docker push ${DOCKER_IMAGE}
                        '''
                    }
                }
            }
        }

        stage('Kubernetes Deployment') {
            steps {
                script {
                    withKubeConfig([credentialsId: 'mymini_cred']) {
                        sh '''
                            kubectl apply -f job_dep.yml
                            kubectl apply -f service.yml
                            kubectl get pods
                            kubectl get deployments
                        '''
                    }
                }
            }
        }
    }
}

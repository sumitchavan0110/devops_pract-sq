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
    }
}

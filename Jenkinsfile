pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    def gitRepoUrl = 'https://github.com/agraharibrijesh/devops_pract.git'
                    def gitCredentialsId = 'github_pat' // Use the GitHub PAT credential ID
                
                    checkout([$class: 'GitSCM', branches: [[name: 'master']], userRemoteConfigs: [[url: gitRepoUrl, credentialsId: gitCredentialsId]])
                }
            }
        }
    }
}

pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                bat './mvnw clean' 
            }
        }
        stage('test') {
            steps {
                bat './mvnw test' 
            }
        }
        stage('package') {
            steps {
                bat './mvnw package' 
            }
        }
        stage('deploy') {
            when 
            {
                expression{ env.BRANCH_NAME == 'master' }
            }
            steps {
                bat './mvnw deploy' 
            }
        }
    }
    post {
        success {
           slackSend message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
       }
       failure {
           slackSend message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
       }
       always {
           slackSend message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
       }
    }
}

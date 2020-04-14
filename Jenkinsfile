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
}

pipeline {
    agent none
    environment {
        BUILD_STATUS = 'FAILED'
        RUN_BISECT = 'FALSE'
    }
    stages {
        stage('master build and test') {
            agent any
            when {
                expression{ env.BRANCH_NAME == 'master' }
            }
            stages {
                script {
                    if (env.LAST_SUCCESS_HASH != 0) {
                        if (env.BUILD_QUEUE_COUNT != 8) {
                            counter++
                        }
                        else {
                            stage('build') {
                                steps {
                                    bat './mvnw clean'
                                }
                            }
                            stage('test') {
                                steps {
                                    bat './mvnw test'
                                }
                            }
                            env.BUILD_QUEUE_COUNT = 0
                        }
                    }
                }
            }
            post{
                success{
                    BUILD_STATUS = 'PASSED'
                }
                failure{
                    BUILD_STATUS = 'FAILED'
                }
            }
        }
        stage('master package') {
            script {
                if (env.BRANCH_NAME == 'master') {
                    if (BUILD_STATUS == 'PASSED') {
                        env.LAST_SUCCESS_HASH = env.GIT_COMMIT
                        steps {
                            bat './mvnw package'
                        }
                    }
                    else {
                        if (env.LAST_SUCCESS_HASH != 0) {
                            echo 'git bisect'
                            RUN_BISECT = 'TRUE'
                        }
                    }
                }
            }
        }
    }
    post {
        success {
           slackSend message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
       failure {
           script {
               if (RUN_BISECT == 'TRUE') {
                    bat "git bisect start ${env.GIT_COMMIT} ${env.LAST_SUCCESS_HASH}"
                    bat "git bisect run mvn clean test"
                    bat "git bisect reset"
               }
           }
           slackSend message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
       }
       always {
           slackSend message: "Build Started - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
       }
    }
}

pipeline {
    agent none
    environment {
        LOCAL_BUILD_STATUS = 'FAILED'
        RUN_BISECT = 'FALSE'
        LAST_SUCCESS_HASH = readFile 'https://github.com/mathiaco/spring-petclinic/blob/master/HASH_FILE'
        BUILD_QUEUE_COUNT = readFile 'https://github.com/mathiaco/spring-petclinic/blob/master/BUILD_QUEUE_COUNT'
    }
    stages {
        stage('master build and test') {
            agent any
            when {
                expression{ env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    echo 'last hash '
                    echo LAST_SUCCESS_HASH
                    if (LAST_SUCCESS_HASH != '0') {
                        if (BUILD_QUEUE_COUNT != 8) {
                            echo 'increment counter currently at '
                            echo BUILD_QUEUE_COUNT
                            echo BUILD_QUEUE_COUNT + 1 > 'https://github.com/mathiaco/spring-petclinic/blob/master/BUILD_QUEUE_COUNT'
                        }
                    }
                    else {
                        echo 'cleaning and testing'
                        bat './mvnw clean'
                        bat './mvnw test'
                        echo 0 > 'https://github.com/mathiaco/spring-petclinic/blob/master/BUILD_QUEUE_COUNT'
                    }
                }
            }
            post {
                success {
                    script {
                        echo 'test passed'
                        LOCAL_BUILD_STATUS = 'PASSED'
                    }
                }
                failure {
                    script {
                        echo 'test failed'
                        LOCAL_BUILD_STATUS = 'FAILED'
                    }
                }
            }
        }
        stage('master package') {
            agent any
            when {
                expression{ env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    if (LOCAL_BUILD_STATUS == 'PASSED') {
                        echo 'will build package'
                        env.LAST_SUCCESS_HASH = env.GIT_COMMIT
                        bat './mvnw package'
                    }
                    else {
                        if (LAST_SUCCESS_HASH != '0') {
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
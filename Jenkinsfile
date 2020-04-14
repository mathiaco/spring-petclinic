pipeline {
    agent none
    environment {
        LOCAL_BUILD_STATUS = 'FAILED'
        RUN_BISECT = 'FALSE'
    }
    stages {
        stage('master build and test') {
            agent any
            when {
                expression{ env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    env.LAST_SUCCESS_HASH = readFile 'HASH_FILE'
                    env.BUILD_QUEUE_COUNT = readFile 'BUILD_QUEUE_COUNT'
                    echo 'last hash '
                    echo LAST_SUCCESS_HASH
                    if (LAST_SUCCESS_HASH.contains('0')) {
                        echo 'cleaning and testing'
                        bat './mvnw clean'
                        bat './mvnw test'
                        writeFile file: 'BUILD_QUEUE_COUNT', text: '0'
                    }
                    else {
                        if (!BUILD_QUEUE_COUNT.contains('8')) {
                            echo 'increment counter currently at '
                            newcount = (BUILD_QUEUE_COUNT as Integer) + 1
                            echo newcount.toString()
                            writeFile file: 'BUILD_QUEUE_COUNT', text: newcount.toString()
                        }
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
                        bat './mvnw package'
                        writeFile file: 'HASH_FILE', text: env.GIT_COMMIT
                    }
                    else {
                        if (LAST_SUCCESS_HASH.contains('0')) {
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
               if (env.RUN_BISECT == 'TRUE') {
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

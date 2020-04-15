pipeline {
    agent none
    environment {
        LOCAL_BUILD_STATUS = 'FAILED'
        // var to indicate if bisect is needed
        RUN_BISECT = 'FALSE'
        RUN_PACKAGE = 'FALSE'
    }
    stages {
        stage('master build and test') {
            agent any
            when {
                expression{ env.BRANCH_NAME == 'master' }
            }
            steps {
                script {
                    // read last successful hash and current build count
                    env.LAST_SUCCESS_HASH = readFile 'HASH_FILE'
                    env.BUILD_QUEUE_COUNT = readFile 'BUILD_QUEUE_COUNT'
                    echo 'last hash '
                    echo LAST_SUCCESS_HASH
                    // if no successful hash is stored, build and test
                    if (LAST_SUCCESS_HASH.contains('NONE')) {
                        echo 'cleaning and testing'
                        bat './mvnw clean'
                        bat './mvnw test'
                        RUN_PACKAGE = 'TRUE' // indicate to run package if test passes
                        // current build count stays 0
                        writeFile file: 'BUILD_QUEUE_COUNT', text: '0'
                    }
                    else {
                        // else if we don't have 8 logs in the queue yet, increment the count and finish
                        if (!BUILD_QUEUE_COUNT.contains('8')) {
                            echo 'increment counter currently at '
                            newcount = (BUILD_QUEUE_COUNT as Integer) + 1
                            echo newcount.toString()
                            writeFile file: 'BUILD_QUEUE_COUNT', text: newcount.toString()
                        }
                        else { // if 8 logs are in the queue, build and test
                            echo 'cleaning and testing'
                            bat './mvnw clean'
                            bat './mvnw test'
                            RUN_PACKAGE = 'TRUE' // indicate to run package if test passes
                            // current build reset 0
                            writeFile file: 'BUILD_QUEUE_COUNT', text: '0'
                        }
                    }
                }
            }
            post {
                success {
                    script {
                        echo 'test passed'
                        // indicate test passed
                        LOCAL_BUILD_STATUS = 'PASSED'
                    }
                }
                failure {
                    script {
                        echo 'test failed'
                        // indicate test failed
                        LOCAL_BUILD_STATUS = 'FAILED'
                        if (RUN_PACKAGE == 'TRUE') {
                            // if good commit is stored and test failed then indicate to git bisect
                            if (!LAST_SUCCESS_HASH.contains('NONE')) {
                                echo 'git bisect'
                                RUN_BISECT = 'TRUE'
                            }
                        }
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
                    // if we build before
                    if (RUN_PACKAGE == 'TRUE') {
                        // test passed
                        if (LOCAL_BUILD_STATUS == 'PASSED') {
                            echo 'will build package'
                            bat './mvnw package'
                            // save current as good commit
                            writeFile file: 'HASH_FILE', text: env.GIT_COMMIT
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

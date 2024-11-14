pipeline {
    agent any

    environment {
        // You can set these directly if necessary
        SFDC_USERNAME = ''
        HUB_ORG = env.HUB_ORG_DH
        SFDC_HOST = env.SFDC_HOST_DH
        JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
        CONNECTED_APP_CONSUMER_KEY = env.CONNECTED_APP_CONSUMER_KEY_DH
    }

    stages {
        stage('Checkout Source') {
            steps {
                checkout scm
            }
        }

        stage('Deploy Code') {
            steps {
                withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
                    script {
                        def rc
                        // Handle Unix vs Windows
                        if (isUnix()) {
                            rc = sh returnStatus: true, script: "${toolbelt} force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile ${jwt_key_file} --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        } else {
                            // On Windows, path to key file should be validated
                            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile C:/Users/Ritika/Downloads/server.key --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                        }

                        if (rc != 0) {
                            error "Hub org authorization failed with exit code ${rc}"
                        }

                        println "Authorization completed with status ${rc}"

                        // Deploy the project
                        def rmsg
                        if (isUnix()) {
                            rmsg = sh returnStdout: true, script: "${toolbelt} project deploy start -d manifest/. -o ${HUB_ORG}"
                        } else {
                            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" project deploy start -d manifest/. -o ${HUB_ORG}"
                        }

                        // Log deployment message
                        println "Deployment message: ${rmsg}"
                    }
                }
            }
        }

        stage('Run Apex Tests') {
            steps {
                script {
                    // Run Apex tests after deployment to validate
                    sh 'sfdx force:apex:test:run --resultformat human --wait 10'
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment was successful!'
        }
        failure {
            echo 'Deployment failed.'
        }
        always {
            // Clean up, notify, etc.
        }
    }
}

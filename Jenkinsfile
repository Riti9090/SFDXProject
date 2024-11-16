pipeline {
    agent any

    environment {
        SF_CONSUMER_KEY = env.SF_CONSUMER_KEY
        SF_USERNAME = env.SF_USERNAME
        SERVER_KEY_CREDENTIALS_ID = env.SERVER_KEY_CREDENTIALS_ID
        TEST_LEVEL = 'RunLocalTests'
        SF_INSTANCE_URL = env.SF_INSTANCE_URL
    }

    stages {
        stage('checkout source') {
            steps {
                checkout scm
            }
        }

        stage('Authorize Org') {
            steps {
                script {
                    def rc = command("${toolbelt}/sf org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwt-key-file ${server_key_file} --set-default-org --alias Dev")
                    if (rc != 0) {
                        error 'Salesforce dev hub org authorization failed.'
                    }
                }
            }
        }

        stage('Deploy and Run Tests') {
            steps {
                script {
                    def rc = command("${toolbelt}/sf project deploy start --wait 10 --manifest/. --targetusername Dev --testlevel ${TEST_LEVEL}")
                    if (rc != 0) {
                        error 'Salesforce deploy and test run failed.'
                    }
                }
            }
        }

        stage('Run Tests In Org') {
            steps {
                script {
                    def rc = command("${toolbelt}/sf apex run test --target-org Dev --wait 10 --result-format tap --code-coverage --test-level ${TEST_LEVEL}")
                    if (rc != 0) {
                        error 'Salesforce unit test run in Dev org failed.'
                    }
                }
            }
        }
    }

    // Define the helper method inside a script block or outside the pipeline block
    post {
        always {
            script {
                def command(script) {
                    echo "Running command: ${script}"
                    if (isUnix()) {
                        return sh(returnStatus: true, script: script)
                    } else {
                        return bat(returnStatus: true, script: script)
                    }
                }
            }
        }
    }
}

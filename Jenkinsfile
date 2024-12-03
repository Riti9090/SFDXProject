import groovy.json.JsonSlurperClassic

node {
    def SF_CONSUMER_KEY_QA = env.SF_CONSUMER_KEY_QA
    def SF_USERNAME_QA = env.SF_USERNAME_QA
    def SERVER_KEY_CREDENTIALS_ID = env.SERVER_KEY_CREDENTIALS_ID
    def TEST_LEVEL = 'RunLocalTests'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL

    def toolbelt = tool 'toolbelt'

    // -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }

    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce JWT key credentials.
    // -------------------------------------------------------------------------
    withEnv(["HOME=${env.WORKSPACE}"]) {

        withCredentials([file(credentialsId: SERVER_KEY_CREDENTIALS_ID, variable: 'server_key_file')]) {

            // -------------------------------------------------------------------------
            // Authorize the Dev Org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize QA') {
                rc = command "\"${toolbelt}\" org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY_QA} --username ${SF_USERNAME_QA} --jwt-key-file ${server_key_file} --alias qa"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Approval Step
            // -------------------------------------------------------------------------

            stage('Approval') {
                input message: 'Do you approve deployment to the Dev Org?',
                      parameters: [
                          string(defaultValue: 'yes', description: 'Approve deployment?', name: 'Approval')
                      ]
            }

            // -------------------------------------------------------------------------
            // Fetch Delta Changes Using Git
            // -------------------------------------------------------------------------

            stage('Identify Delta Changes') {
                // Get the list of changed files using Git diff between the latest commit and the previous commit
                echo 'Identifying changed files...'
                sh '''
                git fetch --all
                git diff --name-only HEAD HEAD~1 > changed_files.txt
                '''
                // Read the changed files into a variable and print them
                def changedFiles = readFile('changed_files.txt').split('\n')
                echo "Changed files: ${changedFiles}"

                // Set environment variable for files to deploy
                env.DEPLOY_FILES = changedFiles.join(' ')
            }

            // -------------------------------------------------------------------------
            // Deploy Changed Files to Salesforce
            // -------------------------------------------------------------------------

            stage('Deploy Delta Changes to Salesforce') {
                echo 'Deploying changed files to Salesforce...'

                // Only deploy files that were changed
                if (env.DEPLOY_FILES != '') {
                    rc = command "\"${toolbelt}\" force:source:deploy -p ${DEPLOY_FILES} --target-org qa --test-level ${TEST_LEVEL} --json --loglevel fatal"
                    if (rc != 0) {
                        error 'Salesforce deployment failed.'
                    }
                } else {
                    echo 'No changes detected. Skipping deployment.'
                }
            }

            // -------------------------------------------------------------------------
            // Push Source to Test Scratch Org (Optional)
            // -------------------------------------------------------------------------

            stage('Push To Test Scratch Org') {
                rc = command "\"${toolbelt}\" project deploy start --target-org qa"
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }
        }
    }
}

def command(script) {
    if (isUnix()) {
        return sh(returnStatus: true, script: script);
    } else {
        return bat(returnStatus: true, script: script);
    }
}

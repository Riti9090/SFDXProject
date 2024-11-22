
import groovy.json.JsonSlurperClassic

node {

	  def SF_CONSUMER_KEY = env.SF_CONSUMER_KEY
    def SF_USERNAME = env.SF_USERNAME
    def SERVER_KEY_CREDENTIALS_ID = env.SERVER_KEY_CREDENTIALS_ID
    def TEST_LEVEL = 'RunLocalTests'
    def SF_INSTANCE_URL = env.SF_INSTANCE_URL
    def SF_CONSUMER_KEY_QA = env.SF_CONSUMER_KEY_QA
    def SF_USERNAME_QA = env.SF_USERNAME_QA

    def toolbelt = tool 'toolbelt'
	
	// -------------------------------------------------------------------------
    // Check out code from source control.
    // -------------------------------------------------------------------------

    stage('checkout source') {
        checkout scm
    }


    // -------------------------------------------------------------------------
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
	withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {
		
		// -------------------------------------------------------------------------
            // Authorize the Dev /' org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize Dev') {
                rc = command "\"${toolbelt}\"  org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwt-key-file ${server_key_file} --alias dev"
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
            // Push source to test scratch org.
            // -------------------------------------------------------------------------

            stage('Push To Test Scratch Org') {
                rc = command "\"${toolbelt}\" project deploy start --target-org dev"
                if (rc != 0) {
                    error 'Salesforce push to test scratch org failed.'
                }
            }
	 // -------------------------------------------------------------------------
            // Approval Step
            // -------------------------------------------------------------------------
            stage('Approval') {
                input message: 'Do you approve deployment to the QA Org?',
                      parameters: [
                          string(defaultValue: 'yes', description: 'Approve deployment?', name: 'Approval')
                      ]
            }
	 // -------------------------------------------------------------------------
            // Push to the 'qa' branch (simplified)
            // -------------------------------------------------------------------------
            stage('Push to QA Branch') {
    script {
        // Ensure all remote branches are fetched
        bat 'git fetch --all'

        // Check if the qa branch exists, if not create and push it
        bat '''
            git branch -r | findstr /C:"origin/qa" >nul
            if %errorlevel% neq 0 (
                echo "Remote branch 'origin/qa' does not exist. Creating it locally and pushing..."
                git checkout -b qa
                git push origin qa
            ) else (
                echo "Switching to existing 'qa' branch..."
                git checkout qa
            )
        '''

        // Stage all changes
        bat 'git add .'

        // Commit the changes, handling the case where there are no changes
        bat 'git commit -m "Your commit message" || echo No changes to commit'

        // Push the changes to the remote qa branch
        bat 'git push origin qa'
    }
}

	// -------------------------------------------------------------------------
            // Authorize the QA org with JWT key and give it an alias.
            // -------------------------------------------------------------------------
            stage('Authorize QA') {
                rc = command "\"${toolbelt}\" org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY_QA} --username ${SF_USERNAME_QA} --jwt-key-file ${server_key_file} --alias qa"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }

            // -------------------------------------------------------------------------
            // Push to QA Org.
            // -------------------------------------------------------------------------
            stage('Push To Test QA Org') {
                rc = command "\"${toolbelt}\" project deploy start --target-org qa"
                if (rc != 0) {
                    error 'Salesforce push to test QA org failed.'
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

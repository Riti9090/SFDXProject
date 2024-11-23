
import groovy.json.JsonSlurperClassic

node {

	   def SF_CONSUMER_KEY_PROD = env.SF_CONSUMER_KEY_PROD
       def SF_USERNAME_PROD = env.SF_USERNAME_PROD
       def SERVER_KEY_CREDENTIALS_ID = env.SERVER_KEY_CREDENTIALS_ID
       def TEST_LEVEL = 'RunLocalTests'
       def SF_INSTANCE_URL_PROD = env.SF_INSTANCE_URL_PROD

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
            // Authorize the PROD /' org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize PROD') {
                rc = command "\"${toolbelt}\"  org login jwt --instance-url ${SF_INSTANCE_URL_PROD} --client-id ${SF_CONSUMER_KEY_PROD} --username ${SF_USERNAME_PROD} --jwt-key-file ${server_key_file} --alias prod"
                if (rc != 0) {
                    error 'Salesforce PROD hub org authorization failed.'
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

            stage('Push To Production Org') {
                rc = command "\"${toolbelt}\" project deploy start --target-org prod"
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

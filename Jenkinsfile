
import groovy.json.JsonSlurperClassic

node {

	   def SF_CONSUMER_KEY_QA = env.SF_CONSUMER_KEY
       def SF_USERNAME_QA = env.SF_USERNAME
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
    // Run all the enclosed stages with access to the Salesforce
    // JWT key credentials.
    // -------------------------------------------------------------------------
    
	withEnv(["HOME=${env.WORKSPACE}"]) {
        
        withCredentials([file(credentialsId: SERVER_KEY_CREDENTALS_ID, variable: 'server_key_file')]) {
		
		// -------------------------------------------------------------------------
            // Authorize the Dev /' org with JWT key and give it an alias.
            // -------------------------------------------------------------------------

            stage('Authorize QA') {
                rc = command "\"${toolbelt}\"  org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY_QA} --username ${SF_USERNAME_QA} --jwt-key-file ${server_key_file} --alias qa"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }
			
	      // -------------------------------------------------------------------------
            // Push source to test scratch org.
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

#!groovy

import groovy.json.JsonSlurperClassic

node {

    def SF_CONSUMER_KEY=env.SF_CONSUMER_KEY
    def SF_USERNAME=env.SF_USERNAME
    def SERVER_KEY_CREDENTALS_ID=env.SERVER_KEY_CREDENTALS_ID
    def TEST_LEVEL='RunLocalTests'
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
            // Authorize the  org with JWT key and give it an alias.
            // -------------------------------------------------------------------------
			
			 stage('Authorize Org') {
			 
				rc = command "${toolbelt}/sf org login jwt --instance-url ${SF_INSTANCE_URL} --client-id ${SF_CONSUMER_KEY} --username ${SF_USERNAME} --jwt-key-file ${server_key_file} --set-default-org --alias Dev"
                if (rc != 0) {
                    error 'Salesforce dev hub org authorization failed.'
                }
            }
			
			// -------------------------------------------------------------------------
		   // Deploy metadata and execute unit tests.
		  // -------------------------------------------------------------------------

		    stage('Deploy and Run Tests') {
		    rc = command "${toolbelt}/sf project deploy start --wait 10 --manifest/.  --targetusername Dev --testlevel ${TEST_LEVEL}"
		    if (rc != 0) {
			error 'Salesforce deploy and test run failed.'
		        }
		    }
			
			// -------------------------------------------------------------------------
			// Run unit tests in test scratch org.
			// -------------------------------------------------------------------------

			stage('Run Tests In Org') {
			rc = command "${toolbelt}/sf apex run test --target-org Dev --wait 10 --result-format tap --code-coverage --test-level ${TEST_LEVEL}"
			if (rc != 0) {
			error 'Salesforce unit test run in Dev org failed.'
				}
			}
	
    def command(script) {
       if (isUnix()) {
         return sh(returnStatus: true, script: script);
       } else {
		        return bat(returnStatus: true, script: script);
      }
	
			

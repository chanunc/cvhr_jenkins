#!groovy

pipeline {
	agent any

	parameters {
		string(name: 'CVHR_BRANCH', defaultValue: 'staging', description: 'CiviHR branch to build with CiviCRM-Buildkit')
		string(name: 'CVHR_SITENAME', defaultValue: 'hr17', description: 'CiviHR site name')
	}

	environment {
		WEBROOT = "/opt/buildkit/build/${params.CVHR_SITENAME}"
		CVHR_EXT_ROOT = "$WEBROOT/sites/all/modules/civicrm/tools/extensions/civihr"
		WEBURL = "http://jenkins.compucorp.co.uk:8900"
	}

  	stages {
    	// TODO: Consider destroy site before or after build
	    stage('Pre-tasks execution') {
	      steps {
				// DEBUG: print environment vars
				sh 'printenv | sort'

				// Destroy existing site
				sh "civibuild destroy ${params.CVHR_SITENAME} || true"

				// Test build tools
				sh 'amp test'
			}
	    }

	    // TODO: Parameterise; buildName, branchName
	    stage('Build site') {
			steps {
				// build site with CiviCRM-Buildkit
				sh """
				  civibuild create ${params.CVHR_SITENAME} --type hr16 --civi-ver 4.7.18 --hr-ver ${params.CVHR_BRANCH} --url $WEBURL --admin-pass 1234
				"""

				// Copy and Replace PR commit code base to CVHR_EXT_ROOT
				sh """
				  rsync -av --delete $WORKSPACE/ $CVHR_EXT_ROOT/

				  cd $WEBROOT
				  drush civicrm-upgrade-db
				  drush cvapi extension.upgrade
				"""

				// DEBUG: Verify latest PR commit on CVHR_EXT_ROOT
				sh """
				  echo 'Verify latest PR commit'
				  cd $CVHR_EXT_ROOT
				  git log --oneline -n2 
				"""
			}
	    }

	    /* Testing PHP */
	    // TODO: Shared env; webRootPath
	    // TODO: Test report after all tests
	    stage('Test PHP') {
			steps {
				script {
					// Get civihr extensions list
					def extensions = listCivihrExtensions()
					
					// Execute PHP test
					for (int i = 0; i<extensions.size(); i++) {
						testPHPUnit(extensions[i])
					}
				}
			}
	    }
		
		/* Testing JS */
	    stage('Testing JS: Install NPM in parallel') {
			steps {
				script {
					// Get civihr extensions list
					def extensions = listCivihrExtensions()
					def extensionTestings = [:]

					// Install NPM jobs
					for (int i = 0; i<extensions.size(); i++) {
						def index = i
						extensionTestings[extensions[index]] = {
						  echo "Installing NPM: " + extensions[index]
						  installNPM(extensions[index])
						}
					}
					// Running install NPM jobs in parallel
					parallel extensionTestings
				}
			}
	    }
	    // TODO: Execute test and Generate report without stop on fail
	    stage('Testing JS: Test JS in sequent') {
			steps {
				script {
					def extensions = listCivihrExtensions()
					def extensionTestings = [:]

					// Testing JS in sequent
					for (int j = 0; j<extensions.size(); j++) {
						def index = j
						echo "Testing with Gulp: " + extensions[index]
						testJS(extensions[index])  
					}
				}
			}
	    }
  	}
}

/* Execute PHPUnit testing
 * params: extensionName
 */
def testPHPUnit(String extensionName){
	sh """
		cd $CVHR_EXT_ROOT/${extensionName}
		phpunit4 || true
	"""
}
/* Installk JS Testing
 * params: extensionName
 */
def installNPM(String extensionName){
	sh """
		cd $CVHR_EXT_ROOT/${extensionName}
		npm install || true
	"""
}
/* Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
	sh """
		cd $CVHR_EXT_ROOT/${extensionName}
		gulp test || true
	"""
}
/* Get list of enabled CiviHR extensions
 * CVAPI - drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print $NF}' | sort
 */
def listCivihrExtensions(){
	// All enabled cvhr extensions
	// return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort").split("\n")

	// Select just some enabled cvhr extensions:
	// com.civicrm.hrjobroles, hrjobcontract, org.civicrm.reqangular, uk.co.compucorp.civicrm.hrcore
 	// return sh(returnStdout: true, script: "cd $WEBROOT/${params.CVHR_SITENAME}/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort | grep 'reqangular\\|hrcore\\|hrjobcontract\\|hrjobroles' ").split("\n")
 	
 	// Manually select cvhr extensions
 	return [
 		'uk.co.compucorp.civicrm.hrcore', 
 		'uk.co.compucorp.civicrm.hrleaveandabsences',
 		'com.civicrm.hrjobroles',
 	]
}


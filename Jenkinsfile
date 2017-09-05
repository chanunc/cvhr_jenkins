#!groovy

pipeline {
	agent any

	environment {
		WEBROOT = "/opt/buildkit/build"
		WEBURL = "http://jenkins.compucorp.co.uk:8900"
	}

	parameters {
		string(name: 'CVHR_BRANCH', defaultValue: 'staging', description: 'CiviHR branch to build with CiviCRM-Buildkit')
		string(name: 'CVHR_SITENAME', defaultValue: 'hr17', description: 'CiviHR site name')
	}

  	stages {
    	// TODO: Consider destroy site before or after build
	    stage('Pre-tasks execution') {
	      steps {
	      	
				// DEBUG: print environment vars
				sh 'printenv'

				// Current Branch
				script {
					def currentBranch = getCurrentBranch()
					env.CURRENT_BRANCH = 'CurrentBranch: '+currentBranch

					echo "Current Branch: "+env.CURRENT_BRANCH+", BRANCH_NAME: $BRANCH_NAME"
				}

				// Destroy existing site
				sh "civibuild destroy ${params.CVHR_SITENAME} || true"

				// Test build tools
				sh 'amp test'
			}
	    }

	    // TODO: Parameterise; buildName, branchName
	    stage('Build site') {
			steps {
				// DEBUG: print civihr branch associated with CiviCRM-Buildkit 
				echo "Branch name: ${params.CVHR_BRANCH}"
				echo "Current Branch: "+env.CURRENT_BRANCH+", BRANCH_NAME: $BRANCH_NAME"

				// build site with CiviCRM-Buildkit
				sh """
				  civibuild create ${params.CVHR_SITENAME} --type hr16 --civi-ver 4.7.18 --hr-ver ${params.CVHR_BRANCH} --url $WEBURL --admin-pass c0mpuc0rp
				  cd $WEBROOT/${params.CVHR_SITENAME}
				  drush civicrm-upgrade-db
				  drush cvapi extension.upgrade
				"""
			}
	    }

	    /* Testing PHP */
	    // TODO: Shared env; webRootPath
	    // TODO: Test report after all tests
	    stage('Test PHP') {
			steps {
				echo 'Testing PHP'

				script {
					// Get civihr extensions list
					def extensions = listCivihrExtensions()

					for (int i = 0; i<extensions.size(); i++) {
						// Execute PHP test
						testPHPUnit(extensions[i])
					}
				}
			}
	    }

	    /* Testing JS */
	    // TODO: Execute test and Generate report without stop on fail
	    stage('Test JS Parallel') {
			steps {
				echo 'Testing JS Parallel'

				script{
					// Get civihr extensions list
					def extensions = listCivihrExtensions()
					def extensionTestings = [:]

					// Install NPM jobs in parallel
					for (int i = 0; i<extensions.size(); i++) {
						def index = i
						extensionTestings[extensions[index]] = {
						  echo "Installing NPM: " + extensions[index]
						  installNPM(extensions[index])
						}
					}

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
		cd $WEBROOT/${params.CVHR_SITENAME}/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
		phpunit4 || true
	"""
}
/* Installk JS Testing
 * params: extensionName
 */
def installNPM(String extensionName){
	sh """
		cd $WEBROOT/${params.CVHR_SITENAME}/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
		npm install || true
	"""
}
/* Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
	sh """
		cd $WEBROOT/${params.CVHR_SITENAME}/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
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
/* Get current branch name
 */
def getCurrentBranch() {
 	// return sh(returnStdout: true, script: "cd $WORKSPACE; git rev-parse --abbrev-ref HEAD")
	def issueNo = sh(returnStdout: true, script: "cd $WORKSPACE; git log --format=%B -n 1 | awk -F'[/:]' '{print \$1}'").trim()
	return sh(returnStdout: true, script: "cd $WORKSPACE; git branch --all | grep ${issueNo} | awk -F '[//]' '{print \$3}'").trim()
}

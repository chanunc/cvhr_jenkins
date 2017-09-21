#!groovy

pipeline {
	agent any

	parameters {
		string(name: 'CVHR_BRANCH', defaultValue: 'staging', description: 'CiviHR branch to build with CiviCRM-Buildkit')
		string(name: 'CVHR_BUILDNAME', defaultValue: "hr17-dev_$BRANCH_NAME", description: 'CiviHR site name')
		booleanParam(name: 'DESTORY_SITE', defaultValue: false, description: 'Destroy built site after build finish')
	}

	environment {
		WEBROOT = "/opt/buildkit/build/${params.CVHR_BUILDNAME}"
		CVCRM_EXT_ROOT = "$WEBROOT/sites/all/modules/civicrm/tools/extensions"
		DR_MODU_ROOT = "$WEBROOT/sites/all/modules"
		WEBURL = "http://jenkins.compucorp.co.uk:8900"
	}

  	stages {
	    stage('Pre-tasks execution') {
	      steps {
				// DEBUG: print all environment vars
				sh 'printenv | sort'

				// Destroy existing site
				sh "civibuild destroy ${params.CVHR_BUILDNAME} || true"

				// Test build tools
				sh 'amp test'
			}
	    }

	    stage('Build site') {
			steps {
				echo 'Build site with CV Buildkit'
				// Build site with CV Buildkit
				sh """
				  civibuild create ${params.CVHR_BUILDNAME} --type hr16 --civi-ver 4.7.18 --hr-ver ${params.CVHR_BRANCH} --url $WEBURL --admin-pass 1234
				"""
				
				script {
					// Change git remote of civihr ext
					changeCivihrGitRemote()

					// Get repos & branch name
					def prBranch = getCurrentBranch()
					def envBranch = params.CVHR_BRANCH
					if (prBranch.startsWith("hotfix-")) {
						envBranch = 'master'
					}

					// Checkout PR Branch in CiviHR repos
					echo 'Checking out CiviHR repos..'
					sh """
						cd $CVCRM_EXT_ROOT
						git-scan foreach -c \"git checkout -b testing-${prBranch} --track remotes/origin/${prBranch}\" || true
					"""

					// Merge PR Branch in CiviHR repos
					def cvhrRepos = listCivihrRepos()
					for (int i=0; i<cvhrRepos.size(); i++) {
						tokens = cvhrRepos[i].tokenize('/');
						echo 'Merging ' + tokens[tokens.size()-1]
						try {
							sh """
								cd ${cvhrRepos[i]}
								git merge origin/${envBranch} --no-edit
							"""	
						} catch (err) {
							echo "Something failed at Check out PR Branch in CiviHR extension: ${cvhrRepos[i]}"
							echo "Failed: ${err}"
						}
					}

					// Upgrade Drupal & CiviCRM extensions
					echo 'Upgrade Drupal & CV extensions'
					sh """
						cd $WEBROOT
						drush features-revert civihr_employee_portal_features -y
	    				drush features-revert civihr_default_permissions -y
	    				drush updatedb -y
	    				drush cvapi extenion.upgrade -y
	    				drush cc all
	    				drush cc civicrm
    				"""
				}
			}
	    }

	    /* Testing PHP */
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
			post {
                always {
                	// XUnit
					step([
	                    $class: 'XUnitBuilder',
                    	thresholds: [
	                    	[$class: 'FailedThreshold',
	                          failureNewThreshold: '5',
	                          failureThreshold: '5',
	                          unstableNewThreshold: '1',
	                          unstableThreshold: '1'],
	                        [$class: 'SkippedThreshold',
	                          failureNewThreshold: '0',
	                          failureThreshold: '0',
	                          unstableNewThreshold: '0',
	                          unstableThreshold: '0']
                    	],
	                    tools: [[$class: 'JUnitType', pattern: 'reports/phpunit/*.xml']]
	                ])
            	}
            }
	    }
		
		/* Testing JS */
		// TODO: Execute test and Generate report without stop on fail
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
						  echo 'Installing NPM: ' + extensions[index]
						  installNPM(extensions[index])
						}
					}
					// Running install NPM jobs in parallel
					parallel extensionTestings
				}
			}
	    }
	    stage('Testing JS: Test JS in sequent') {
			steps {
				script {
					// Get civihr extensions list
					def extensions = listCivihrExtensions()
					def extensionTestings = [:]

					// Testing JS in sequent
					for (int j = 0; j<extensions.size(); j++) {
						def index = j
						echo 'Testing with Gulp: ' + extensions[index]
						testJS(extensions[index])  
					}
				}
			}
	    }
  	}

    post {
    	always {
    		// Destroy built site
    		script {
				if (params.DESTORY_SITE == true) {
					echo 'Destroying built site...'
					sh "civibuild destroy ${params.CVHR_BUILDNAME} || true"
				}
    		}
    	}
    }
}
/*
 *	Change URL Git remote of civihr main repositry to the URL where configured by Jenkins project
 */
def changeCivihrGitRemote() {
	echo 'Changing Civihr git URL..'
	def pulledCvhrRepo = sh(returnStdout: true, script: "cd $WORKSPACE; git remote -v | grep fetch | awk '{print \$2}'").trim()
	sh """
		cd $CVCRM_EXT_ROOT/civihr
		git remote set-url origin ${pulledCvhrRepo}
		git fetch --all
	"""
}
/* 
 *	Get current branch name
 */
def getCurrentBranch() {
	echo 'Get current branch...'
 	// return sh(returnStdout: true, script: "cd $WORKSPACE; git rev-parse --abbrev-ref HEAD")
	def issueNo = sh(returnStdout: true, script: "cd $WORKSPACE; git log --format=%B -n 1 | awk -F'[/:]' '{print \$1}'").trim()
	def currentBranchName = sh(returnStdout: true, script: "cd $WORKSPACE; git ls-remote origin | grep ${issueNo} | awk -F '[//]' '{print \$NF}'").trim()
	
	echo "issueNo: ${issueNo} | Branch: ${currentBranchName}"
	return currentBranchName
}
/* 
 * Execute PHPUnit testing
 * params: extensionName
 */
def testPHPUnit(String extensionName){
	def extensionShortName = extensionName.tokenize('.')[-1]
	echo "PHPUnit testing: ${extensionShortName}" 
	sh """
		cd $CVCRM_EXT_ROOT/civihr/${extensionName}
		phpunit4 \
			--log-junit $WORKSPACE/reports/phpunit/result-phpunit_${extensionShortName}.xml \
			--coverage-html $WORKSPACE/reports/phpunit/resultPHPUnitHtml_${extensionShortName} \
			|| true
	"""
}
/* 
 * Installk JS Testing
 * params: extensionName
 */
def installNPM(String extensionName){
	sh """
		cd $CVCRM_EXT_ROOT/civihr/${extensionName}
		npm install || true
	"""
}
/* 
 * Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
	echo "JS Testing ${extensionName.tokenize('.')[-1]}"
	sh """
		cd $CVCRM_EXT_ROOT/civihr/${extensionName}
		gulp test || true
	"""
}
/* 
 * Get a list of CiviHR repository
 * https://compucorp.atlassian.net/wiki/spaces/PCHR/pages/68714502/GitHub+repositories
 */
def listCivihrRepos(){
	// Get list of CiviHR repository paths using git-scan
	return sh(returnStdout: true, script: "cd $WEBROOT; git-scan foreach -c 'pwd' | grep civihr").split("\n")

	// Manually set the list
	// return [
	// 	"$CVCRM_EXT_ROOT/civihr",
	// 	"$CVCRM_EXT_ROOT/civihr_tasks",
	// 	"$CVCRM_EXT_ROOT/org.civicrm.shoreditch",
	// 	"$CVCRM_EXT_ROOT/org.civicrm.styleguide",
	// 	"$DR_MODU_ROOT/civihr-custom/civihr_employee_portal",
	// ]
}
/* 
 * Get a list of enabled CiviHR extensions
 */
def listCivihrExtensions(){
	// All enabled cvhr extensions
	// return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort").split("\n")

	// Select just some enabled cvhr extensions:
	// com.civicrm.hrjobroles, hrjobcontract, org.civicrm.reqangular, uk.co.compucorp.civicrm.hrcore
 	// return sh(returnStdout: true, script: "cd $WEBROOT/${params.CVHR_BUILDNAME}/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort | grep 'reqangular\\|hrcore\\|hrjobcontract\\|hrjobroles' ").split("\n")

 	// Manually select cvhr extensions
 	return [
 		'uk.co.compucorp.civicrm.hrcore',
 		'uk.co.compucorp.civicrm.hrleaveandabsences',
 		'com.civicrm.hrjobroles',
 	]
}


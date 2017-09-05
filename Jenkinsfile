#!groovy

pipeline {
	agent any	

	// TODO: Trigger job by Github PR
	// triggers {
	//     upstream 'project-name,other-project-name', hudson.model.Result.SUCCESS
	// }

  parameters {
  	string(name: 'CVHR_BRANCH', defaultValue: 'staging', description: 'CiviHR git repo branch to build')
  	string(name: 'CVHR_SITENAME', defaultValue: 'hr17', description: 'CiviHR site name')
  }

  stages {
    // TODO: Consider destroy site before or after build
    stage('Pre-tasks execution') {
      steps {
      	
      	// DEBUG: print environment vars
      	sh 'printenv'

      	script {

      		// Current Branch
			def currentBranch = getCurrentBranch()
			env.CURRENT_BRANCH = currentBranch

			echo 'Current Branch: '+currentBranch
			echo "BRANCH_NAME: $BRANCH_NAME"
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
      	// echo "Branch name: $BRANCH_NAME"
      	// echo "Branch name: $CURRENT_BRANCH"
      	echo "Branch name: {params.CVHR_BRANCH}"

        sh """
          civibuild create ${params.CVHR_SITENAME} --type hr16 --civi-ver 4.7.18 --hr-ver {params.CVHR_BRANCH} --url http://jenkins.compucorp.co.uk:8900 --admin-pass c0mpuc0rp
          cd /opt/buildkit/build/hr17/sites/
          drush civicrm-upgrade-db
          drush cvapi extension.upgrade
        """
      }
    }

    // TODO: Shared env; webRootPath
    // TODO: Test report after all tests
    stage('Test PHP') {
      steps {
        echo 'Testing PHP'

        script {
          // Get the list of cvivihr extensions to test
          def extensions = listEnabledCivihrExtensions()

          for (int i = 0; i<extensions.size(); i++) {
            // Execute PHP test
            testPHPUnit(extensions[i])
          }
        }

        publishers {
          textFinder(/^FAILURES!$/, '', true, false, true)
        }
        // publishers {
        //   /* Add textFinder from Job DSL plugin
        //    */
        //   // textFinder(String regularExpression
        //   // , String fileSet = ''
        //   // , boolean alsoCheckConsoleOutput = false
        //   // , boolean succeedIfFound = false
        //   // , unstableIfFound = false
          
        //   textFinder(/^FAILURES!$/, '', true, true, false)
        // }
      }
    }

    // TODO: Execute test and Generate report without stop on fail
   
    // stage('Test JS'){
    //   steps{
    //     echo 'Testing JS'

    //     script{
    //       // Get the list of cvivihr extensions to test
    //       def extensions = listEnabledCivihrExtensions()

    //       for (int i = 0; i<extensions.size(); i++) {
    //       // Execute JS test
    //       testJS(extensions[i])
    //       }
    //     }
    //   }
    // }
    
    /* Parallel Test JS
     */
    stage('Test JS Parallel') {
      steps {
        echo 'Testing JS Parallel'

        script{
          // get extensions list
          def extensions = listEnabledCivihrExtensions()
          def extensionTestings = [:]

          // Parallel Install NPM jobs 
          for (int i = 0; i<extensions.size(); i++) {
            def index = i
            extensionTestings[extensions[index]] = {
              echo "Installing NPM: " + extensions[index]
              installJS(extensions[index])
            }
          }
          parallel extensionTestings

          // Sequenctially test JS
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
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    phpunit4 || true
  """
}
/* Installk JS Testing
 * params: extensionName
 */
def installJS(String extensionName){
  sh """
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    npm install || true
  """
}
/* Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
  sh """
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    gulp test || true
  """
}
/* Get list of enabled CiviHR extensions
 * CVAPI - drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print $NF}' | sort
 */
def listEnabledCivihrExtensions(){
  echo 'Get list of enabled CiviHR extensions'
  // All cvhr extensions
  // return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort").split("\n")
  
  /* Some cvhr extensions:
   *  com.civicrm.hrjobroles
   *  hrjobcontract
   *  org.civicrm.reqangular
   *  uk.co.compucorp.civicrm.hrcore
   */
  return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort | grep 'reqangular\\|hrcore\\|hrjobcontract\\|hrjobroles' ").split("\n")
}
/* Get current branch name
 */
def getCurrentBranch() {
  // return sh(returnStdout: true, script: "cd $WORKSPACE; git rev-parse --abbrev-ref HEAD")
  def issueNo = sh(returnStdout: true, script: "cd $WORKSPACE; git log --format=%B -n 1 | awk -F'[/:]' '{print \$1}'").trim()
  return sh(returnStdout: true, script: "cd $WORKSPACE; git branch --all | grep ${issueNo} | awk -F '[//]' '{print \$3}'").trim()
}

#!groovy

pipeline {
  agent any
  
  parameters {
  	string(name: 'CVHR_BRANCH', defaultValue: '1.7-wip', description: 'CiviHR git repo branch to build')
  }

  stages {
    // TODO: Consider destroy site before or after build
    stage('Pre-tasks execution') {
      steps {
      	
      	// DEBUG: print environment vars
      	// sh 'printenv'

        // Destroy existing site
        sh "civibuild destroy $BRANCH_NAME"

        // Test build tools
        sh 'amp test'
      }
    }

    // TODO: Parameterise; buildName, branchName
    stage('Build site') {
      steps {
      	echo "Sitename: $BRANCH_NAME"

        sh """
          civibuild create $BRANCH_NAME --type hr16 --civi-ver 4.7.18 --hr-ver ${params.CVHR_BRANCH} --url http://jenkins.compucorp.co.uk:8900 --admin-pass c0mpuc0rp
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

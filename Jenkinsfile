#!groovy
pipeline {
  agent any
  
  stages {
    stage('Pre-tasks execution') {
      // TODO: Consider destroy site before or after build
      steps {
        echo 'Destroy existing site'
        sh 'civibuild destroy hr17'
        echo 'Test build tools'
        sh 'amp test'
      }
    }

    stage('Build site') {
      // TODO: Parameterise; buildName, branchName
      steps {
        sh '''
          civibuild create hr17 --type hr16 --civi-ver 4.7.18 --hr-ver 1.7-wip --url http://jenkins.compucorp.co.uk:8900 --admin-pass c0mpuc0rp
          cd /opt/buildkit/build/hr17/sites/
          drush civicrm-upgrade-db
          drush cvapi extension.upgrade
        '''
      }
    }

    // TODO: Shared env; webRootPath
    // TODO: Test report after all tests
    stage('Test PHP') {
      steps {
        echo 'Testing PHP'

        script{
          // Get the list of cvivihr extensions to test
          def extensions = listEnabledCivihrExtensions()

          for (int i = 0; i<extensions.size(); i++) {
            // Execute PHP test
            testPHPUnit(extensions[i])
          }
        }

        publishers {
          /* Add textFinder from Job DSL plugin
           */
          // textFinder(String regularExpression
          // , String fileSet = ''
          // , boolean alsoCheckConsoleOutput = false
          // , boolean succeedIfFound = false
          // , unstableIfFound = false
          textFinder(/^FAILURES!$/, '', true, true, false)
          // textFinder(/^OK ($/, '', true, true, false)
        }
      }
    }

    // TODO: Parallell npm install every extensions
    stage('Test JS'){
      steps{
        echo 'Testing JS'
        script{
          // Get the list of cvivihr extensions to test
          def extensions = listEnabledCivihrExtensions()

          // TODO: Execute test and Generate report without stop on fail
          for (int i = 0; i<extensions.size(); i++) {
            // Execute JS test
            testJS(extensions[i])
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
/* Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
  sh """
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    npm install
    gulp test || true
  """
}
/* Get list of enabled CiviHR extensions
 * CVAPI - drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print $NF}' | sort
 */
def listEnabledCivihrExtensions(){
  echo 'Get list of enabled CiviHR extensions'
  return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort").split("\n")
}

#!groovy

// Get the list of cvivihr extensions to test
def extensions = listEnabledCivihrExtensions()

pipeline {
  agent any
  
  stages {
    stage('Test build tools') {
      steps {
        sh 'amp test'
      }
    }

    stage('Build site') {
      steps {
        // TODO: Parameterise; buildName, branchName
        sh '''
          civibuild create hr17 --type hr16 --civi-ver 4.7.18 --hr-ver 1.7-wip --url http://jenkins.compucorp.co.uk:8900 --admin-pass c0mpuc0rp
          cd /opt/buildkit/build/hr17/sites/
          drush civicrm-upgrade-db
          drush cvapi extension.upgrade
        '''
      }
    }

    stage('Test PHP') {
      steps {
        // TODO: Shared env; webRootPath
        // TODO: Execute test and Generate report without stop on fail
        for (int i = 0; i<extensions.size(); i++) {
          // Execute PHP test
          testPHPUnit(extensions[i])
        }
      }
    }

    stage('Test JS'){
      steps{
        // TODO: Execute test and Generate report without stop on fail
        for (int i = 0; i<extensions.size(); i++) {
          // Execute JS test
          testJS(extensions[i])
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
    phpunit4
  """
}
/* Execute JS Testing
 * params: extensionName
 */
def testJS(String extensionName){
  sh """
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    npm install
    gulp test
  """
}
/* Get list of enabled extensions in extensions/civihr folder
 * CVAPI - drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print $NF}' | sort
 */
def listEnabledCivihrExtensions(){
  return sh(returnStdout: true, script: "cd /opt/buildkit/build/hr17/sites/; drush cvapi extension.get statusLabel=Enabled return=path | grep '/civihr/' | awk -F '[//]' '{print \$NF}' | sort").split("\n")
}


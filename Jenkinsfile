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
        testExtensionWithPHPUnit("uk.co.compucorp.civicrm.hrcore")
        testExtensionWithPHPUnit("hrjobcontract")
      }
    }
    stage('Test JS'){
      steps{
        sh '''
          cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/org.civicrm.reqangular
          npm install
          gulp test
        '''
      }
    }
  }
}

// Execute PHPUnit testing by extension
def testExtensionWithPHPUnit(String extensionName){
  sh """
    cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/${extensionName}
    phpunit4
  """
}
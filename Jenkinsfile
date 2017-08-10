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
        sh '''
          cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/uk.co.compucorp.civicrm.hrcore
          phpunit4
        '''
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

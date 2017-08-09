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
        // sh 'civibuild create hr16 --civi-ver 4.7.18 --hr-ver staging --url http://jenkins.compucorp.co.uk:8901 --admin-pass c0mpuc0rp'
        sh 'civibuild create hr17 --type hr16 --civi-ver 4.7.18 --hr-ver 1.7-wip --url http://jenkins.compucorp.co.uk:8900 --admin-pass c0mpuc0rp'
      }
    }
    stage('Test PHP') {
      steps {
        sh '''cd /opt/buildkit/build/hr17/sites/all/modules/civicrm/tools/extensions/civihr/uk.co.compucorp.civicrm.hrcore
phpunit4'''
      }
    }
  }
}

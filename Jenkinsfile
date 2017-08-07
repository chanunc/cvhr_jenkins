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
        sh 'civibuild create hr16 --civi-ver 4.7.18 --hr-ver staging --url http://jenkins.compucorp.co.uk:8901 --admin-pass c0mpuc0rp'
      }
    }
    stage('Test PHP') {
      steps {
        sh '''cd /opt/buildkit/build/hr16/sites/all/modules/civicrm/tools/extensions/civihr/uk.co.compucorp.civicrm.hrcore
phpunit4'''
      }
    }
  }
}
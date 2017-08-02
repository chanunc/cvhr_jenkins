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
        sh 'civibuild create hr16 --civi-ver 4.7.18 --hr-ver staging --url http://jenkin.compucorp.co.uk:8901 --admin-pass c0mpuc0rp'
      }
    }
  }
}
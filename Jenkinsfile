pipeline {
  agent {
    label "mvn"
  }
  parameters {
    booleanParam(name: 'RELEASE', defaultValue: false, description: 'Should we release a version ?')
  }
  stages {
    stage("build") {
      when {
        expression { return !params.RELEASE }
      }
      steps {
        container('atlas-sdk') {
          script {
            sh 'whoami'
//            sh 'ssh-keyscan -t rsa github.com >> ~/.ssh/known_hosts'
//            sh 'cat ~/.ssh/known_hosts'
            sh 'atlas-mvn -B clean deploy -s /usr/src/mvn/settings.xml'
          }
        }
      }
    }
    stage("release") {
      when {
        expression { return params.RELEASE }
      }
      steps {
        container('atlas-sdk') {
          script {
            sh 'atlas-mvn -B clean release:prepare -s /usr/src/mvn/settings.xml'
            sh 'atlas-mvn -B release:perform -s /usr/src/mvn/settings.xml'
          }
        }
      }
    }
  }
}

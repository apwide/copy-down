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
        container('mvn-node') {
          script {
            sh 'mvn -B clean deploy -s /usr/src/mvn/settings.xml'
          }
        }
      }
    }
    stage("release") {
      when {
        expression { return params.RELEASE }
      }
      steps {
        container('mvn-node') {
          script {
            sh "whoami"
            sh 'id'
            sshagent(credentials: ['ci-github-ssh-credentials']) {
              sh 'mvn -B clean release:prepare -s /usr/src/mvn/settings.xml -DpushChanges=false'
              sh 'mvn -B release:perform -s /usr/src/mvn/settings.xml -DlocalCheckout=true'

              // script just to register Host (avoid key verification on git push) but github returns Exit ccode 1
              // because no shell supported, that's why we register and ignore statusCode
              def statusCode = sh script: 'ssh -vT -o "StrictHostKeyChecking=no" git@github.com', returnStatus: true
              sh 'git push origin $(git rev-parse --abbrev-ref HEAD)' // git push to remote branch
              sh 'git push --tags' // push all created tags
            }
          }
        }
      }
    }
  }
}

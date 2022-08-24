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
            sh 'id'
            sh 'cat /etc/passwd'
            sleep 60
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
            sh "whoami"
            sh 'id'
            sshagent(credentials: ['ci-github-ssh-credentials']) {
//              sh 'ssh-add -L'
//              sh 'ssh -vT -o "StrictHostKeyChecking=no" git@github.com'
              sh "whoami"
              def statusCode = sh script: 'ssh -vT -o "StrictHostKeyChecking=no" git@github.com', returnStatus: true
              sh 'atlas-mvn -B clean release:prepare -s /usr/src/mvn/settings.xml -DpushChanges=false'
              sh 'atlas-mvn -B release:perform -s /usr/src/mvn/settings.xml -DlocalCheckout=true'

              sh 'git push origin $(git rev-parse --abbrev-ref HEAD)' // git push to remote branch
              sh 'git push --tags' // push all created tags

            }
          }
        }
      }
    }
  }
}

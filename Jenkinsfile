pipeline {
    agent {
      label "jenkins-jx-base"
    }
    environment {
      ORG               = 'maddalimanideep'
      APP_NAME          = 'sfdemo-209'
      CHARTMUSEUM_CREDS = credentials('jenkins-x-chartmuseum')
    }
    stages {
      stage('Build Release') {
        when {
          branch env.BRANCH_NAME
        }
        steps {
          container('jx-base') {
            // ensure we're not on a detached head
            sh "git checkout ${env.BRANCH_NAME}"
            sh 'git config --global credential.username maddalimanideep'
            sh "git config --global credential.helper store"
            sh "jx step validate --min-jx-version 1.1.73"
            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
          }
          dir ('./charts/sfdemo-209') {
            container('jx-base') {
              sh "make tag"
            }
          }
        }
      }
      stage('Promote to Environments') {
        when {
          branch env.BRANCH_NAME
        }
        steps {
          dir ('./charts/sfdemo-209') {
            container('jx-base') {
              // release the helm chart
              sh 'jx step helm release'

              // promote through all 'Auto' promotion Environments
              // sh 'jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
              //sh 'jx step helm apply --namespace=jx-staging --name=sfdemo-209 --no-helm-version=true --wait=false'
              sh 'helm3 upgrade sfdemo-209 . --namespace=jx-staging --force --install'
            }
          }
        }
      }
    }
    post {
        always {
            cleanWs()
        }
    }
  }

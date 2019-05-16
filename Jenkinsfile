pipeline {
    agent {
      label "jenkins-jx-base"
    }
    environment {
      ORG               = 'tnahak'
      APP_NAME          = 'mysql-11'
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
            sh 'git config --global credential.username tnahak'
            sh "git config --global credential.helper store"
            sh "jx step validate --min-jx-version 1.1.73"
            sh "jx step git credentials"
            // so we can retrieve the version in later steps
            sh "echo \$(jx-release-version) > VERSION"
          }
          dir ('./charts/mysql-11') {
            container('jx-base') {
              sh "make tag"
            }
          }
        }
      }
      stage('Parallel In Sequential') {
                    parallel {
                        stage('In Parallel 1') {
                            steps {
                                echo "In Parallel 1"
                            }
                        }
                        stage('In Parallel 2') {
                            steps {
                                echo "In Parallel 2"
                            }
                        }
                    }
      stage('Promote to Environments') {
        when {
          branch env.BRANCH_NAME
        }
        steps {
          dir ('./charts/mysql-11') {
            container('jx-base') {
              // release the helm chart
              sh 'jx step helm release'

              // promote through all 'Auto' promotion Environments
              // sh 'jx promote -b --all-auto --timeout 1h --version \$(cat ../../VERSION)'
              sh 'jx step helm apply --namespace=jxtitu --name=mysql-11 --no-helm-version=true --wait=false'
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

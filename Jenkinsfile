def job_name = env.JOB_NAME.split('/')[0]

pipeline {
  agent {
    kubernetes {
      label "build-${job_name}"
      defaultContainer 'node'
      containerTemplate {
        name 'node'
        image 'node:lts'
        ttyEnabled true
        command 'cat'
      }
    }
  }
  stages {
    stage("Checkout repo") {
      steps {
        checkout(scm)
      }
    }
    stage("Install dependencies") {
      steps {
        sh "yarn"
      }
    }
    stage("Analyze code") {
      parallel {
        stage('unit-tests') {
          steps {
            sh "yarn test"
          }
        }
      }
    }
    stage("Build") {
      parallel {
        stage("Build Package") {
          stages {
            stage("NPM publish") {
              when { branch 'master' }
              steps {
                withCredentials([file(credentialsId: 'npm-publish', variable: 'SecretFile')]) {
                  sh "cp $SecretFile ~/.npmrc"
                }
                sh "npm publish"
              }
            }
          }
        }
      }
    }
  }
}

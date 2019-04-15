pipeline {
  agent {
    node {
      label 'maven'
    }

  }
  stages {
    stage('Build') {
      parallel {
        stage('Build') {
          steps {
            echo 'Building new2'
            sh 'printenv'
          }
        }
        stage('') {
          steps {
            git(url: 'https://github.com/b05020412//jenkins-demo', branch: 'master', credentialsId: 'ss', poll: true)
          }
        }
      }
    }
    stage('Test') {
      steps {
        echo 'Testing'
      }
    }
    stage('Deploy - Staging') {
      steps {
        echo 'Deploy - Staging'
      }
    }
    stage('Sanity check') {
      steps {
        input 'Does the staging environment look ok?'
      }
    }
    stage('Deploy - Production') {
      steps {
        echo 'Deploy - Production updated 20190409'
      }
    }
  }
  environment {
    DISABLE_AUTH = 'false'
    DB_ENGINE = 'sqlite'
  }
  post {
    always {
      echo 'One way or another, I have finished'
      deleteDir()

    }

    success {
      echo 'I succeeeded!'

    }

    unstable {
      echo 'I am unstable :/'

    }

    failure {
      mail(to: 'team1@example.com', subject: 'Failed Pipeline: ', body: 'Something is wrong with ')

    }

    changed {
      echo 'Things were different before...'

    }

  }
}
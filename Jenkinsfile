pipeline {
    def inventory_file = ''
    agent {
        label 'sslave-172-31-25-236'
    }
    
    environment {
        DISABLE_AUTH = 'false'
        DB_ENGINE    = 'sqlite'
    }
    
    stages {
        stage('Build') {
            steps {
                echo 'Building new2'
                sh 'printenv'       
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
                input "Does the staging environment look ok?"
            }
        }

        stage('Deploy - Production') {
            steps {
                echo 'Deploy - Production updated 20190409'
            }
        }
        

   
    }
    
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            mail to: 'team1@example.com',
             subject: "Failed Pipeline: ",
             body: "Something is wrong with "
        }
        changed {
            echo 'Things were different before...'
        }
    }
}

pipeline {
    agent any

    stages {

        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
            }
        }

        stage('CMD Test') {
            steps {
                echo 'Testing Windows CMD access'
                bat 'echo CMD WORKING'
                bat 'where cmd'
            }
        }

        stage('Build') {
            steps {
                echo 'Building package...'
            }
        }
    }
}

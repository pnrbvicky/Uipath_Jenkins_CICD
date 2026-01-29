pipeline {
    agent any

    environment {
        // UiPath CLI path (update if different)
        UIPATH_CLI = "${WORKSPACE}/CLI/cached/UiPath.CLI.Windows/25.10.3/tools/uipcli.dll"

        // Package output folder
        PACKAGE_OUTPUT = "${WORKSPACE}/Packages"

        // Project version (example)
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}-b${BUILD_ID}"
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building project with version ${PACKAGE_VERSION}"

                script {
                    // Detect OS and use correct command
                    if (isUnix()) {
                        sh "dotnet \"${UIPATH_CLI}\" pack \"${WORKSPACE}\" --output \"${PACKAGE_OUTPUT}\" --version ${PACKAGE_VERSION}"
                    } else {
                        bat "dotnet \"${UIPATH_CLI}\" pack \"${WORKSPACE}\" --output \"${PACKAGE_OUTPUT}\" --version ${PACKAGE_VERSION}"
                    }
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                // Add your UiPath test commands here, if any
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying to UAT..."
                // Add deployment steps here
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying to Production..."
                // Add deployment steps here
            }
        }
    }

    post {
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            cleanWs()
            echo "Workspace cleaned"
        }
    }
}

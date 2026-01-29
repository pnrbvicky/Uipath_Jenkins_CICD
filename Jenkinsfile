pipeline {
    agent any

    environment {
        WORKSPACE_DIR = "${env.WORKSPACE}"
        PACKAGE_OUTPUT = "${env.WORKSPACE}/Packages"
        UIPATH_CLI = "${env.WORKSPACE}/CLI/cached/UiPath.CLI.Windows/25.10.3/tools/uipcli.dll"
        TIMESTAMP = "${new Date().format('yyyyMMddHHmmss')}"
        PACKAGE_VERSION = "1.0.${BUILD_NUMBER}-b${TIMESTAMP}"
    }

    options {
        timeout(time: 1, unit: 'HOURS') // pipeline timeout
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Checking out source code..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/pnrbvicky/Uipath_Jenkins_CICD.git',
                        credentialsId: 'GitCred'
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                echo "Building project with version ${PACKAGE_VERSION}"

                // Make sure output folder exists
                bat "mkdir \"${PACKAGE_OUTPUT}\""

                // Run UiPath CLI pack command
                bat """
                dotnet "${UIPATH_CLI}" pack "${WORKSPACE_DIR}" --output "${PACKAGE_OUTPUT}" --version ${PACKAGE_VERSION}
                """
            }
        }

        stage('Test') {
            steps {
                echo "Running tests..."
                // Example: run test workflows via CLI or call test scripts
                // bat "dotnet \"${UIPATH_CLI}\" test ..."
                echo "Tests skipped (add your test commands here)"
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying package to UAT..."
                // Example CLI deploy command
                // bat "dotnet \"${UIPATH_CLI}\" deploy --package \"${PACKAGE_OUTPUT}/YourPackage.nupkg\" --environment UAT"
                echo "Deploy to UAT skipped (add your deploy commands here)"
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying package to Production..."
                // Example CLI deploy command
                // bat "dotnet \"${UIPATH_CLI}\" deploy --package \"${PACKAGE_OUTPUT}/YourPackage.nupkg\" --environment Production"
                echo "Deploy to Production skipped (add your deploy commands here)"
            }
        }
    }

    post {
        always {
            cleanWs()
            echo "Pipeline finished. Job URL: ${env.BUILD_URL}"
        }
        failure {
            echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        }
    }
}

pipeline {
    agent any

    environment {
        WORKSPACE_DIR = "${env.WORKSPACE}"
        UIPATH_PROJECT = "MainProject"          // name of your UiPath project folder
        PACKAGE_OUTPUT = "${env.WORKSPACE}/Packages" // where the .nupkg will go
        VERSION = "1.0.${env.BUILD_NUMBER}"     // dynamic versioning
    }

    stages {

        stage('Checkout SCM') {
            steps {
                echo "Checking out Git repository..."
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: 'https://github.com/pnrbvicky/Uipath_Jenkins_CICD.git',
                        credentialsId: 'GitCred'
                    ]]
                ])
            }
        }

        stage('Build / Pack') {
            steps {
                echo "Packing UiPath project..."
                script {
                    UiPathPack(
                        path: "${WORKSPACE_DIR}/${UIPATH_PROJECT}",  // full path to project folder
                        version: "${VERSION}",                        // version string
                        outputPath: "${PACKAGE_OUTPUT}"               // optional: output folder
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running Tests (if any)..."
                // Add UiPathTest step here if you have automated tests
                // e.g., UiPathTest(path: "${WORKSPACE_DIR}/${UIPATH_PROJECT}")
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying to UAT..."
                // Add UiPathDeploy step here for UAT
                // e.g., UiPathDeploy(packagePath: "${PACKAGE_OUTPUT}/${UIPATH_PROJECT}.${VERSION}.nupkg", environment: "UAT")
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying to Production..."
                // Add UiPathDeploy step here for Production
                // e.g., UiPathDeploy(packagePath: "${PACKAGE_OUTPUT}/${UIPATH_PROJECT}.${VERSION}.nupkg", environment: "Production")
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline succeeded!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

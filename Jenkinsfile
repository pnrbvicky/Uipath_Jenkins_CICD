pipeline {
    agent any

    environment {
        // Set any environment variables here
        UIPATH_PROJECT = 'MainProject'
        WORKSPACE_DIR = "C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main"
        PACKAGE_OUTPUT = "${WORKSPACE_DIR}/Packages"
        PROJECT_FOLDER = "${WORKSPACE_DIR}/${UIPATH_PROJECT}"
    }

    stages {

        stage('Checkout SCM') {
            steps {
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
                        project: env.UIPATH_PROJECT,
                        folder: env.PROJECT_FOLDER,
                        output: env.PACKAGE_OUTPUT,
                        version: [
                            major: 1,
                            minor: 0,
                            patch: env.BUILD_NUMBER.toInteger()
                        ]
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running UiPath Tests..."
                // Example: add UiPathTest step if needed
                // UiPathTest(project: env.UIPATH_PROJECT, folder: env.PROJECT_FOLDER)
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying to UAT..."
                // Example: add UiPathDeploy step if needed
                // UiPathDeploy(environment: 'UAT', package: "${PACKAGE_OUTPUT}/${UIPATH_PROJECT}.nupkg")
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploying to Production..."
                // Example: add UiPathDeploy step if needed
                // UiPathDeploy(environment: 'Production', package: "${PACKAGE_OUTPUT}/${UIPATH_PROJECT}.nupkg")
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
    }
}

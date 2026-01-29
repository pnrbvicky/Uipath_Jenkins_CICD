pipeline {
    agent any

    environment {
        // Make sure UiPath CLI is in PATH
        PATH = "C:\\Program Files (x86)\\UiPath\\Studio\\UiPath;${env.PATH}"

        // Set project and output paths
        PROJECT_PATH = "${WORKSPACE}"
        OUTPUT_PATH = "${WORKSPACE}\\Packages"

        // Version for the package
        VERSION = "1.0.${BUILD_NUMBER}-b${BUILD_ID}"
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building UiPath project version ${env.VERSION}"
                script {
                    // Use correct UiPathPack parameters
                    uipathPack(
                        projectPath: env.PROJECT_PATH,
                        outputPath: env.OUTPUT_PATH,
                        version: env.VERSION,
                        publish: true // optional, publish package after build
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo 'Running tests...'
                script {
                    // Run UiPath test (modify parameters as needed)
                    uipathTest(
                        projectPath: env.PROJECT_PATH,
                        robotType: 'Unattended'
                    )
                }
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo 'Deploying package to UAT environment...'
                script {
                    uipathDeploy(
                        packagePath: "${env.OUTPUT_PATH}\\${env.VERSION}.nupkg",
                        environment: 'UAT',
                        orchestratorUrl: 'https://platform.uipath.com/',
                        tenant: 'YourTenantName',
                        folder: 'UAT_Folder',
                        credentialsId: 'OrchestratorCreds'
                    )
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: "Approve deployment to Production?"
                echo 'Deploying package to Production environment...'
                script {
                    uipathDeploy(
                        packagePath: "${env.OUTPUT_PATH}\\${env.VERSION}.nupkg",
                        environment: 'Production',
                        orchestratorUrl: 'https://platform.uipath.com/',
                        tenant: 'YourTenantName',
                        folder: 'Production_Folder',
                        credentialsId: 'OrchestratorCreds'
                    )
                }
            }
        }
    }

    post {
        success { echo 'Pipeline completed successfully!' }
        failure { echo 'Pipeline failed!' }
        always { cleanWs() }
    }
}

pipeline {
    agent any

    environment {
        // Versioning
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator
        UIPATH_ORCH_URL = 'https://cloud.uipath.com'
        UIPATH_ORCH_TENANT = 'DefaultTenant'
        UIPATH_FOLDER = 'UnAttended'

        // Jenkins Credential ID
        UIPATH_CRED = 'APIUserKey'
    }

    options {
        timeout(time: 60, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Code checked out"
            }
        }

        stage('Pack') {
            steps {
                script {
                    echo "Build No : ${env.BUILD_NUMBER}"

                    UiPathPack(
                        projectJsonPath: 'project.json',
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        version: [
                            $class: 'ManualVersionEntry',
                            version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.0"
                        ],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        stage('Deploy') {
            steps {
                UiPathDeploy(
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT}",
                    credentials: "${UIPATH_CRED}",
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    folderName: "${UIPATH_FOLDER}",
                    environments: "${UIPATH_FOLDER}",           // Environment name (can be same as folder)
                    entryPointPaths: 'Main.xaml',               // Entry point file
                    createProcess: true,                        // Create process if not exists
                    traceLevel: 'None'
                )
            }
        }
    }

    post {
        success {
            echo '✅ Successfully deployed to Orchestrator'
        }
        failure {
            echo '❌ Deployment failed'
        }
        always {
            cleanWs()
        }
    }
}

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

        // ----------------------
        // Checkout Stage
        // ----------------------
        stage('Checkout') {
            steps {
                checkout scm
                echo "Code checked out"
            }
        }

        // ----------------------
        // Pack Stage
        // ----------------------
        stage('Pack') {
            steps {
                script {
                    echo "Build No : ${env.BUILD_NUMBER}"

                    UiPathPack(
                        projectJsonPath: 'project.json',
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        version: [
                            $class: 'ManualVersionEntry',
                            version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.0"  // numeric version
                        ],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        // ----------------------
        // Deploy to Orchestrator
        // ----------------------
        stage('Deploy') {
            steps {
                UiPathDeploy(
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT}",
                    credentials: "${UIPATH_CRED}",
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    folderName: "${UIPATH_FOLDER}",
                    traceLevel: 'None'
                )
            }
        }
    }

    // ----------------------
    // Post Actions
    // ----------------------
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

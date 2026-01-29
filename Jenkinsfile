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
                echo "✅ Code checked out"
            }
        }

        stage('Pack') {
            steps {
                script {
                    def gitHash = bat(
                        script: 'git rev-parse --short HEAD',
                        returnStdout: true
                    ).trim()

                    echo "Git Hash: ${gitHash}"
                    echo "Build No: ${env.BUILD_NUMBER}"

                    UiPathPack(
                        projectJsonPath: 'project.json',
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        version: [
                            $class: 'ManualVersionEntry',
                            version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.${gitHash}"
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
                    credentials: credentials("${UIPATH_CRED}"),  // ✅ Corrected
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    folderName: "${UIPATH_FOLDER}",
                    environments: "${UIPATH_FOLDER}",
                    entryPointPaths: 'Main.xaml',
                    createProcess: true,
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

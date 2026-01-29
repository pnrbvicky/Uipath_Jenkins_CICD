pipeline {
    agent any

    environment {
        // Versioning
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator
        UIPATH_ORCH_URL = 'https://cloud.uipath.com'
        UIPATH_ORCH_TENANT = 'DefaultTenant'
        UIPATH_FOLDER = 'UnAttended'           // Folder in Orchestrator
        UIPATH_ENV = 'UnAttended'              // Robot Environment name (must match Orchestrator)

        // Jenkins Credential ID for Orchestrator API
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
                    // Get short Git hash
                    def gitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()

                    echo "Git Hash : ${gitHash}"
                    echo "Build No : ${env.BUILD_NUMBER}"

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
                    credentials: "${UIPATH_CRED}",
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    folderName: "${UIPATH_FOLDER}",       // ✅ Mandatory
                    environments: "${UIPATH_ENV}",        // ✅ Must match Orchestrator environment
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml',
                    createProcess: true
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

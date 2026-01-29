pipeline {
    agent any

    environment {
        // Version control
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator config
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"      // your account logical name
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"    // tenant name
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"        // modern folder
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
    }

    stages {

        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building UiPath package..."

                UiPathPack(
                    projectJsonPath: "project.json",
                    outputPath: "Output",
                    version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                    useOrchestrator: false,
                    traceLevel: 'None'
                )
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying to UiPath Orchestrator..."

                UiPathDeploy(
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    packagePath: "Output",
                    credentials: Token(
                        accountName: "${UIPATH_ORCH_LOGICAL_NAME}",
                        credentialsId: "APIUserKey"
                    ),
                    createProcess: true,
                    entryPointPaths: "Main.xaml",
                    traceLevel: 'None'
                )
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully"
        }
        failure {
            echo "❌ Build failed"
        }
        always {
            cleanWs()
        }
    }
}

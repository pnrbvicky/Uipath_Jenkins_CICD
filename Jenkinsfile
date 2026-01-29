pipeline {
    agent any

    environment {
        MAJOR = '1'
        MINOR = '0'

        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME  = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME  = "UnAttended"
    }

    stages {

        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Branch    : ${env.BRANCH_NAME}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                UiPathPack(
                    projectJsonPath: "project.json",
                    outputPath: "Output\\${env.BUILD_NUMBER}",
                    version: [
                        $class: 'ManualVersionEntry',
                        version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"
                    ],
                    useOrchestrator: false,
                    traceLevel: 'None'
                )
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",

                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",

                    environments: 'DEV',          // REQUIRED
                    createProcess: true,           // 🔥 FIX FOR YOUR ERROR

                    credentials: Token(
                        accountName: "${UIPATH_ORCH_LOGICAL_NAME}",
                        credentialsId: 'APIUserKey'
                    ),

                    entryPointPaths: 'Main.xaml',
                    traceLevel: 'None'
                )
            }
        }
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    post {
        success {
            echo "✅ UiPath deployment successful"
        }
        failure {
            echo "❌ Build failed"
        }
        always {
            cleanWs()
        }
    }
}

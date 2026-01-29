pipeline {
    agent any

    environment {
        MAJOR = '1'
        MINOR = '0'
        // Orchestrator Services - update as per your Orchestrator
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"
    }

    stages {

        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}/${env.BRANCH_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Building package..."
                script {
                    def gitHash = bat(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    UiPathPack(
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        projectJsonPath: "project.json",
                        version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.${gitHash}"],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running tests (if any)..."
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package to Orchestrator..."
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'DEV', // change to your environment if needed
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml',
                    createProcess: true
                )
            }
        }

        stage('Deploy to Production') {
            steps {
                echo "Deploy to Production (if needed)"
            }
        }
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
        }
        always {
            cleanWs()
        }
    }
}

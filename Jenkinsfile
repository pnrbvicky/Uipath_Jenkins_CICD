pipeline {
    agent any

    // Environment Variables
    environment {
        MAJOR = '1'
        MINOR = '0'
        // Orchestrator Config - change as per your Orchestrator
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"
    }

    stages {

        // Prepare Stage
        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}/${env.BRANCH_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
                checkout scm
            }
        }

        // Build Stage
        stage('Build') {
            steps {
                echo "Building package..."
                UiPathPack(
                    outputPath: "Output\\${env.BUILD_NUMBER}",
                    projectJsonPath: "project.json",
                    version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                    useOrchestrator: false,
                    traceLevel: 'None'
                )
            }
        }

        // Test Stage
        stage('Test') {
            steps {
                echo "Running tests (if any)..."
            }
        }

        // Deploy Stage
        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package to Orchestrator..."
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    createProcess: true,                   // mandatory for first deployment
                    environments: 'DEV',                   // mandatory
                    credentials: Token(
                        accountName: "${UIPATH_ORCH_LOGICAL_NAME}",
                        credentialsId: 'APIUserKey'
                    ),
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml'
                )
            }
        }

        // Optional: Deploy to Production
        stage('Deploy to Production') {
            steps {
                echo "Deploy to Production (if needed)"
            }
        }
    }

    // Pipeline Options
    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    // Post Actions
    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo "❌ Deployment failed for Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        }
        always {
            cleanWs()
        }
    }
}

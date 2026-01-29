pipeline {
    agent any

    // Environment Variables
    environment {
        MAJOR = '1'
        MINOR = '0'
        // Orchestrator Services
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"  // replace with your Orchestrator logical name
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant" // replace with your tenant name
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"    // replace with your folder name
    }

    stages {

        // Preparing stage
        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}/${env.BRANCH_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
                checkout scm
            }
        }

        // Build stage
        stage('Build') {
            steps {
                echo "Building package..."

                // Get short Git hash to create unique version
                script {
                    def gitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    env.PACKAGE_VERSION = "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.${gitHash}"

                    echo "Package version: ${env.PACKAGE_VERSION}"

                    UiPathPack(
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        projectJsonPath: "project.json",
                        version: [$class: 'ManualVersionEntry', version: "${env.PACKAGE_VERSION}"],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        // Test stage
        stage('Test') {
            steps {
                echo "Running tests (if any)..."
            }
        }

        // Deploy to Orchestrator
        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package to Orchestrator..."
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'DEV',  // Orchestrator environment
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml'
                )
            }
        }

        // Deploy to Production (optional)
        stage('Deploy to Production') {
            steps {
                echo "Deploy to Production (if needed)"
            }
        }
    }

    // Options
    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    // Post actions
    post {
        success {
            echo "✅ Deployment completed successfully!"
        }
        failure {
            echo "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        }
        always {
            cleanWs()
        }
    }
}

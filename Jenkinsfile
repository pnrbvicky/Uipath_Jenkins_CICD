pipeline {
    agent any

    // Environment Variables
    environment {
        MAJOR = '1'
        MINOR = '0'
        // Orchestrator Services - update as per your config
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "yourOrchestratorUsername"
        UIPATH_ORCH_TENANT_NAME = "yourTenantName"
        UIPATH_ORCH_FOLDER_NAME = "Default"
    }

    stages {

        // ----------------------
        // Prepare Stage
        // ----------------------
        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Workspace : ${env.WORKSPACE}"
                checkout scm
            }
        }

        // ----------------------
        // Build Stage
        // ----------------------
        stage('Build') {
            steps {
                echo "Building package..."
                script {
                    // Get short Git hash (Windows)
                    def gitHash = bat(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    echo "Git hash: ${gitHash}"

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

        // ----------------------
        // Test Stage
        // ----------------------
        stage('Test') {
            steps {
                echo "Running tests (if any)..."
            }
        }

        // ----------------------
        // Deploy to Orchestrator
        // ----------------------
        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package to Orchestrator..."
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    environments: 'UAT', // Update with your actual environment
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml',
                    createProcess: true
                )
            }
        }

        // ----------------------
        // Deploy to Production (optional)
        // ----------------------
        stage('Deploy to Production') {
            steps {
                echo "Deploy to Production (if needed)"
            }
        }

    } // stages

    // ----------------------
    // Pipeline Options
    // ----------------------
    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    // ----------------------
    // Post Actions
    // ----------------------
    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo "❌ FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.JOB_DISPLAY_URL})"
        }
        always {
            cleanWs()
        }
    }

} // pipeline

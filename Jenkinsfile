pipeline {
    agent { label 'windows' } // UiPath requires Windows agent

    // Environment Variables
    environment {
        MAJOR = '1'
        MINOR = '0'
        // Orchestrator Services
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"
        OUTPUT_PATH = "Output\\${env.BUILD_NUMBER}"
    }

    options {
        timeout(time: 80, unit: 'MINUTES') // max pipeline time
        skipDefaultCheckout() // we handle checkout manually
    }

    stages {

        // 1️⃣ Preparing Stage
        stage('Preparing') {
            steps {
                echo "Jenkins Home: ${env.JENKINS_HOME}"
                echo "Jenkins URL: ${env.JENKINS_URL}"
                echo "Job Number: ${env.BUILD_NUMBER}"
                echo "Job Name: ${env.JOB_NAME}"
                echo "Branch Name: ${env.BRANCH_NAME}"

                checkout scm
            }
        }

        // 2️⃣ Build Stage
        stage('Build') {
            steps {
                echo "Building project in workspace: ${env.WORKSPACE}"
                script {
                    UiPathPack(
                        outputPath: "${OUTPUT_PATH}",
                        projectJsonPath: "project.json",
                        version: [$class: 'ManualVersionEntry', version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        // 3️⃣ Test Stage
        stage('Test') {
            steps {
                echo "Testing workflow..."
                // Add real test commands here if needed
            }
        }

        // 4️⃣ Deploy to UAT
        stage('Deploy to UAT') {
            steps {
                echo "Deploying ${BRANCH_NAME} to UAT"
                script {
                    UiPathDeploy(
                        packagePath: "${OUTPUT_PATH}",
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        environments: 'DEV',
                        credentials: [$class: 'TokenAuthenticationEntry', accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'],
                        createProcess: true,
                        traceLevel: 'None',
                        entryPointPaths: 'Main.xaml'
                    )
                }
            }
        }

        // 5️⃣ Deploy to Production
        stage('Deploy to Production') {
            steps {
                echo "Deploying ${BRANCH_NAME} to Production"
                script {
                    UiPathDeploy(
                        packagePath: "${OUTPUT_PATH}",
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        environments: 'PROD',
                        credentials: [$class: 'TokenAuthenticationEntry', accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'],
                        createProcess: true,
                        traceLevel: 'None',
                        entryPointPaths: 'Main.xaml'
                    )
                }
            }
        }
    }

    post {
        success {
            echo "Deployment completed successfully!"
        }
        failure {
            echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_DISPLAY_URL})"
        }
        always {
            cleanWs()
        }
    }
}

pipeline {
    agent any

    // Environment Variables
    environment {
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator (Cloud)
        UIPATH_ORCH_URL = 'https://cloud.uipath.com'
        UIPATH_ORCH_LOGICAL_NAME = 'devellwmqjpn'
        UIPATH_ORCH_TENANT_NAME = 'DefaultTenant'
        UIPATH_ORCH_FOLDER_NAME = 'UnAttended'
    }

    stages {

        stage('Preparing') {
            steps {
                echo "Jenkins Home : ${env.JENKINS_HOME}"
                echo "Jenkins URL  : ${env.JENKINS_URL}"
                echo "Build Number: ${env.BUILD_NUMBER}"
                echo "Job Name    : ${env.JOB_NAME}"
                echo "Branch Name : ${env.BRANCH_NAME}"

                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo "Packaging UiPath project..."

                UiPathPack(
                    outputPath: "Output\\${env.BUILD_NUMBER}",
                    projectJsonPath: "project.json",
                    version: [
                        $class: 'ManualVersionEntry',
                        version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}"
                    ],
                    useOrchestrator: false,
                    traceLevel: 'None'
                )
            }
        }

        stage('Test') {
            steps {
                echo 'Testing stage (placeholder)'
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying package to UiPath Orchestrator..."

                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorUrl: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    credentials: Token(
                        accountName: "${UIPATH_ORCH_LOGICAL_NAME}",
                        credentialsId: 'APIUserKey'
                    ),
                    entryPointPaths: 'Main.xaml',
                    createProcess: true,
                    traceLevel: 'None'
                )
            }
        }

        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Production deployment placeholder'
            }
        }
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    post {
        success {
            echo 'UiPath deployment completed successfully!'
        }
        failure {
            echo "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        always {
            cleanWs()
        }
    }
}

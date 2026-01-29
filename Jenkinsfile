pipeline {
    agent { label 'windows' }

    environment {
        MAJOR = '1'
        MINOR = '0'
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"
    }

    stages {
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

        stage('Build') {
            steps {
                script {
                    // Create a unique version using BUILD_NUMBER + timestamp
                    def timestamp = new Date().format("yyyyMMddHHmmss")
                    env.UNIQUE_VERSION = "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}.${timestamp}"
                    echo "Building project with version: ${env.UNIQUE_VERSION}"

                    UiPathPack(
                        outputPath: "Output\\${env.BUILD_NUMBER}",
                        projectJsonPath: "project.json",
                        version: [$class: 'ManualVersionEntry', version: env.UNIQUE_VERSION],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Testing workflow..."
            }
        }

        stage('Deploy to UAT') {
            steps {
                script {
                    echo "Deploying ${BRANCH_NAME} to UAT"
                    UiPathDeploy(
                        packagePath: "Output\\${env.BUILD_NUMBER}",
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        environments: 'DEV',
                        credentials: [$class: 'TokenAuthenticationEntry', accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'],
                        createProcess: true,
                        traceLevel: 'None',
                        entryPointPaths: 'Main.xaml'
                        // overwrite: true  // optional if you want to replace old package
                    )
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    echo "Deploying ${BRANCH_NAME} to Production"
                    UiPathDeploy(
                        packagePath: "Output\\${env.BUILD_NUMBER}",
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                        folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                        environments: 'PROD',
                        credentials: [$class: 'TokenAuthenticationEntry', accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'],
                        createProcess: true,
                        traceLevel: 'None',
                        entryPointPaths: 'Main.xaml'
                        // overwrite: true  // optional
                    )
                }
            }
        }
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    post {
        success {
            echo 'Deployment completed successfully!'
        }
        failure {
            echo "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_DISPLAY_URL})"
        }
        always {
            cleanWs()
        }
    }
}

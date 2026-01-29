pipeline {
    agent any

    environment {
        MAJOR = '1'
        MINOR = '0'
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "devellwmqjpn"
        UIPATH_ORCH_TENANT_NAME = "DefaultTenant"
        UIPATH_ORCH_FOLDER_NAME = "UnAttended"
    }

    stages {

        stage('Prepare') {
            steps {
                echo "Job       : ${env.JOB_NAME}"
                echo "Build No  : ${env.BUILD_NUMBER}"
                echo "Branch    : ${env.BRANCH_NAME}"
                checkout scm

                // Get short git hash for versioning
                script {
                    GIT_HASH = sh(script: "git rev-parse --short HEAD", returnStdout: true).trim()
                    VERSION = "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}-${GIT_HASH}"
                    echo "Computed Version: ${VERSION}"
                }
            }
        }

        stage('Update project.json') {
            steps {
                // Update version inside project.json
                powershell """
                \$json = Get-Content project.json | ConvertFrom-Json
                \$json.version = '${VERSION}'
                \$json | ConvertTo-Json -Depth 10 | Set-Content project.json
                """
            }
        }

        stage('Build') {
            steps {
                echo "Packing project with version: ${VERSION}"
                UiPathPack(
                    outputPath: "Output\\${env.BUILD_NUMBER}",
                    projectJsonPath: "project.json",
                    version: [$class: 'ManualVersionEntry', version: "${VERSION}"],
                    useOrchestrator: false,
                    traceLevel: 'None'
                )
            }
        }

        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package ${VERSION} to Orchestrator"
                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",
                    createProcess: true,
                    credentials: Token(accountName: "${UIPATH_ORCH_LOGICAL_NAME}", credentialsId: 'APIUserKey'),
                    traceLevel: 'None',
                    entryPointPaths: 'Main.xaml'
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
            echo "❌ Deployment FAILED for '${env.JOB_NAME} [${env.BUILD_NUMBER}]'"
        }
        always {
            cleanWs()
        }
    }
}

pipeline {
    agent any

    environment {
        // Versioning
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator (Cloud)
        UIPATH_ORCH_URL = "https://cloud.uipath.com/"
        UIPATH_ORCH_LOGICAL_NAME = "anupaminc"          // change as per your org
        UIPATH_ORCH_TENANT_NAME  = "Descriptify"        // change as per your tenant
        UIPATH_ORCH_FOLDER_NAME  = "Default"            // change as per your folder
    }

    stages {

        /* =======================
           PREPARE
        ======================= */
        stage('Prepare') {
            steps {
                echo "Jenkins Home       : ${env.JENKINS_HOME}"
                echo "Jenkins URL        : ${env.JENKINS_URL}"
                echo "Job Name           : ${env.JOB_NAME}"
                echo "Build Number       : ${env.BUILD_NUMBER}"
                echo "Git Branch         : ${env.BRANCH_NAME}"

                checkout scm
            }
        }

        /* =======================
           BUILD (PACK)
        ======================= */
        stage('Build') {
            steps {
                echo "Packing UiPath project..."

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

        /* =======================
           TEST (OPTIONAL)
        ======================= */
        stage('Test') {
            steps {
                echo "No automated tests configured yet"
            }
        }

        /* =======================
           DEPLOY TO DEV / UAT
        ======================= */
        stage('Deploy to Orchestrator') {
            steps {
                echo "Deploying package to UiPath Orchestrator..."

                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_URL}",
                    orchestratorTenant: "${UIPATH_ORCH_TENANT_NAME}",
                    folderName: "${UIPATH_ORCH_FOLDER_NAME}",

                    // REQUIRED (do not remove)
                    environments: 'DEV',

                    credentials: Token(
                        accountName: "${UIPATH_ORCH_LOGICAL_NAME}",
                        credentialsId: 'APIUserKey'
                    ),

                    entryPointPaths: 'Main.xaml',
                    traceLevel: 'None'
                )
            }
        }

        /* =======================
           PROD (MANUAL LATER)
        ======================= */
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo "Production deployment can be added here"
            }
        }
    }

    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    post {
        success {
            echo "✅ UiPath CI/CD pipeline completed successfully"
        }
        failure {
            echo "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        always {
            cleanWs()
        }
    }
}

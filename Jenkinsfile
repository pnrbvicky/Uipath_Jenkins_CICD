pipeline {
    agent any

    /*************** ENVIRONMENT VARIABLES ****************/
    environment {
        // Versioning
        MAJOR = '1'
        MINOR = '0'

        // UiPath Orchestrator (Cloud)
        UIPATH_ORCH_ADDRESS = 'https://cloud.uipath.com'
        UIPATH_ORCH_LOGICAL_NAME = 'devellwmqjpn'
        UIPATH_ORCH_TENANT_NAME = 'DefaultTenant'
        UIPATH_ORCH_FOLDER_NAME = 'UnAttended'
    }

    /*********************** STAGES ************************/
    stages {

        stage('Prepare') {
            steps {
                echo "Jenkins Home   : ${env.JENKINS_HOME}"
                echo "Jenkins URL    : ${env.JENKINS_URL}"
                echo "Job Name       : ${env.JOB_NAME}"
                echo "Build Number   : ${env.BUILD_NUMBER}"
                echo "Branch Name    : ${env.BRANCH_NAME}"

                checkout scm
            }
        }

        stage('Build (Pack UiPath Project)') {
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
                echo 'Test stage placeholder (add UiPath Test Automation if needed)'
            }
        }

        stage('Deploy to UiPath Orchestrator (UAT)') {
            steps {
                echo "Deploying package to UiPath Orchestrator..."

                UiPathDeploy(
                    packagePath: "Output\\${env.BUILD_NUMBER}",
                    orchestratorAddress: "${UIPATH_ORCH_ADDRESS}",
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
                echo 'Production deployment placeholder (add approval gate here)'
            }
        }
    }

    /*********************** OPTIONS ***********************/
    options {
        timeout(time: 80, unit: 'MINUTES')
        skipDefaultCheckout()
    }

    /*********************** POST **************************/
    post {
        success {
            echo 'UiPath CI/CD pipeline completed successfully 🎉'
        }
        failure {
            echo "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
        }
        always {
            cleanWs()
        }
    }
}

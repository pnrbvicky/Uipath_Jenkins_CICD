pipeline {
    agent any

    environment {
        // Example: update according to your system
        UIPATH_ORCHESTRATOR_URL = 'https://your-orchestrator-url'
        UIPATH_ORCHESTRATOR_TENANT = 'your-tenant'
        UIPATH_ORCHESTRATOR_USER = 'your-username'
        UIPATH_ORCHESTRATOR_PASSWORD = 'your-password'
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }

        stage('Build / Pack') {
            steps {
                echo "Packing UiPath project..."
                script {
                    UiPathPack(
                        projectPath: 'C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main/MainProject/MainProject.xaml', 
                        outputPath: 'C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main/Packages',
                        version: '1.0.0-b${BUILD_NUMBER}'
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Running UiPath Tests..."
                script {
                    UiPathTest(
                        projectPath: 'C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main/MainProject/MainProject.xaml',
                        testSetName: 'AllTests',
                        orchestratorUrl: "${UIPATH_ORCHESTRATOR_URL}",
                        tenant: "${UIPATH_ORCHESTRATOR_TENANT}",
                        username: "${UIPATH_ORCHESTRATOR_USER}",
                        password: "${UIPATH_ORCHESTRATOR_PASSWORD}"
                    )
                }
            }
        }

        stage('Deploy to UAT') {
            steps {
                echo "Deploying to UAT..."
                script {
                    UiPathDeploy(
                        packagePath: 'C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main/Packages/MainProject.1.0.0-b${BUILD_NUMBER}.nupkg',
                        environmentName: 'UAT',
                        orchestratorUrl: "${UIPATH_ORCHESTRATOR_URL}",
                        tenant: "${UIPATH_ORCHESTRATOR_TENANT}",
                        username: "${UIPATH_ORCHESTRATOR_USER}",
                        password: "${UIPATH_ORCHESTRATOR_PASSWORD}"
                    )
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                input message: 'Approve deployment to Production?'
                echo "Deploying to Production..."
                script {
                    UiPathDeploy(
                        packagePath: 'C:/ProgramData/Jenkins/.jenkins/workspace/Uipath_Jenkins_CICD_main/Packages/MainProject.1.0.0-b${BUILD_NUMBER}.nupkg',
                        environmentName: 'Production',
                        orchestratorUrl: "${UIPATH_ORCHESTRATOR_URL}",
                        tenant: "${UIPATH_ORCHESTRATOR_TENANT}",
                        username: "${UIPATH_ORCHESTRATOR_USER}",
                        password: "${UIPATH_ORCHESTRATOR_PASSWORD}"
                    )
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully!"
        }
        failure {
            echo "Pipeline failed!"
        }
        always {
            cleanWs()
        }
    }
}

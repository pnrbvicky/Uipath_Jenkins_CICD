pipeline {
    agent any

    environment {
        // UiPath CLI version
        UIPATH_CLI_VERSION = "Windows.25.10.3"
    }

    options {
        timeout(time: 1, unit: 'HOURS')
        skipDefaultCheckout(true)
    }

    stages {

        stage('Preparing') {
            steps {
                echo "Jenkins Home: ${JENKINS_HOME}"
                echo "Jenkins URL: ${JENKINS_URL}"
                echo "Job Number: ${BUILD_NUMBER}"
                echo "Job Name: ${JOB_NAME}"

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "main"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [],
                    userRemoteConfigs: [[
                        url: "https://github.com/pnrbvicky/Uipath_Jenkins_CICD.git",
                        credentialsId: "GitCred"
                    ]]
                ])
            }
        }

        stage('Build') {
            steps {
                script {
                    // Safe UiPath package version
                    def timestamp = new Date().format("yyyyMMddHHmm")
                    env.PACKAGE_VERSION = "1.0.${BUILD_NUMBER}-b${timestamp}"
                    echo "Building project with version: ${env.PACKAGE_VERSION}"

                    // Pack the UiPath project
                    UiPathPack(
                        projectPath: "${WORKSPACE}",
                        outputPath: "${WORKSPACE}/Packages",
                        version: env.PACKAGE_VERSION,
                        cliVersion: "${UIPATH_CLI_VERSION}"
                    )
                }
            }
        }

        stage('Test') {
            steps {
                echo "Testing workflow..."
                // Add your test steps here if needed
            }
        }

        stage('Deploy to UAT') {
            steps {
                script {
                    echo "Deploying ${env.BRANCH_NAME} to UAT"
                    UiPathDeploy(
                        packagePath: "${WORKSPACE}/Packages",
                        environmentName: "UAT",
                        cliVersion: "${UIPATH_CLI_VERSION}"
                    )
                }
            }
        }

        stage('Deploy to Production') {
            steps {
                script {
                    echo "Deploying ${env.BRANCH_NAME} to Production"
                    UiPathDeploy(
                        packagePath: "${WORKSPACE}/Packages",
                        environmentName: "Production",
                        cliVersion: "${UIPATH_CLI_VERSION}"
                    )
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            echo "Build finished. Job URL: ${BUILD_URL}"
        }
        failure {
            echo "FAILED: Job '${JOB_NAME} [${BUILD_NUMBER}]'"
        }
    }
}

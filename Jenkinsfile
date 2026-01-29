pipeline {
    agent any

    environment {
        ORCHESTRATOR_URL = 'https://cloud.uipath.com'
        ORCHESTRATOR_TENANT = 'DefaultTenant'
        ORCHESTRATOR_FOLDER = 'UnAttended'
        CREDENTIALS_ID = 'APIUserKey'
        ENTRY_POINT = 'Main.xaml'
        ENVIRONMENT_NAME = 'UnAttended'
        PROJECT_PATH = 'C:\\ProgramData\\Jenkins\\.jenkins\\workspace\\Uipath_Jenkins_CICD_main'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/pnrbvicky/Uipath_Jenkins_CICD.git',
                        credentialsId: 'GitCred'
                    ]]
                ])
                echo '✅ Code checked out'
            }
        }

        stage('Pack') {
            steps {
                script {
                    // Capture Git hash safely
                    def gitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Git Hash: ${gitHash}"
                    echo "Build No: ${env.BUILD_NUMBER}"

                    // Prepare package output path (safe Windows path)
                    def packagePath = "${PROJECT_PATH}\\Output\\${env.BUILD_NUMBER}_${gitHash}"
                    echo "Package Path: ${packagePath}"

                    // UiPath Pack
                    UiPathPack(
                        projectJsonPath: "${PROJECT_PATH}\\project.json",
                        outputPath: packagePath,
                        version: [
                            $class: 'ManualVersionEntry',
                            version: "1.0.${env.BUILD_NUMBER}"  // numeric only
                        ],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                    echo "✅ Package created at ${packagePath}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    // Use the same Git hash as in Pack stage
                    def gitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    def packagePath = "${PROJECT_PATH}\\Output\\${env.BUILD_NUMBER}_${gitHash}"

                    UiPathDeploy(
                        orchestratorAddress: env.ORCHESTRATOR_URL,
                        orchestratorTenant: env.ORCHESTRATOR_TENANT,
                        folderName: env.ORCHESTRATOR_FOLDER,
                        environments: env.ENVIRONMENT_NAME,
                        entryPointPaths: env.ENTRY_POINT,
                        packagePath: packagePath,
                        credentials: env.CREDENTIALS_ID,
                        createProcess: true,
                        traceLevel: 'None'
                    )
                    echo "✅ Deployment successful"
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            echo '🧹 Workspace cleaned'
        }
        success {
            echo '🎉 Pipeline completed successfully'
        }
        failure {
            echo '❌ Deployment failed'
        }
    }
}

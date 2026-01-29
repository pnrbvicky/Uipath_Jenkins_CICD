pipeline {
    agent any

    environment {
        MAJOR = 1
        MINOR = 0
        UIPATH_ORCH_URL = 'https://cloud.uipath.com'
        ORCH_TENANT = 'DefaultTenant'
        ORCH_FOLDER = 'UnAttended'
        ORCH_ENV = 'UnAttended'
        ORCH_CRED = 'APIUserKey' // Make sure this is the SelectEntry credential ID, not string
    }

    stages {

        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/main']],
                    userRemoteConfigs: [[
                        url: 'https://github.com/pnrbvicky/Uipath_Jenkins_CICD.git',
                        credentialsId: 'GitCred'
                    ]]
                ])
                echo "✅ Code checked out"
            }
        }

        stage('Pack') {
            steps {
                script {
                    // Get short Git hash
                    def gitHash = bat(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    echo "Git Hash: ${gitHash}"
                    echo "Build No: ${env.BUILD_NUMBER}"

                    // UiPath Pack
                    UiPathPack(
                        projectJsonPath: 'project.json',
                        outputPath: "Output\\${env.BUILD_NUMBER}_${gitHash}",
                        version: [
                            $class: 'ManualVersionEntry',
                            version: "${MAJOR}.${MINOR}.${env.BUILD_NUMBER}" // ✅ numeric only
                        ],
                        useOrchestrator: false,
                        traceLevel: 'None'
                    )
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    UiPathDeploy(
                        orchestratorAddress: "${UIPATH_ORCH_URL}",
                        orchestratorTenant: "${ORCH_TENANT}",
                        folderName: "${ORCH_FOLDER}",
                        environments: [env.ORCH_ENV],             // List of environments
                        entryPointPaths: ['Main.xaml'],           // List of entry points
                        credentials: credentials(ORCH_CRED),     // Must be SelectEntry type in Jenkins
                        packagePath: "Output\\${env.BUILD_NUMBER}_${gitHash}",
                        createProcess: true,
                        traceLevel: 'None'
                    )
                }
            }
        }

    }

    post {
        always {
            cleanWs()
            echo "❌ Deployment finished (success or failure)"
        }
    }
}

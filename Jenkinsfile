pipeline {
    agent any
    
    parameters {
        choice(name: 'ENV', choices: ['dev', 'prod'], description: 'Environment to deploy to')
        string(name: 'FILE_NAME', defaultValue: 'app', description: 'Имя исполняемого файла')
        booleanParam(name: 'RUN_UNIT', defaultValue: true, description: 'Запускать unit тесты')
        booleanParam(name: 'RUN_INTEGRATION', defaultValue: true, description: 'Запускать integration тесты')
    }
    
    stages {
        stage('Print Deploy Info') {
            steps {
                echo "Deploying to ${params.ENV}"
                echo "Executable file name: ${params.FILE_NAME}"
                echo "Run Unit Tests: ${params.RUN_UNIT}"
                echo "Run Integration Tests: ${params.RUN_INTEGRATION}"
            }
        }
        
        stage('Print Info') {
            steps {
                script {
                    sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)"'
                    sh 'echo "Hash: $(git rev-parse HEAD)"'
                    sh 'echo "g++ version: $(g++ --version)"'
                }
            }
        }
        
        stage('Build Executable file') {
            steps {
                script {
                    sh "g++ app.cpp -o ${params.FILE_NAME}"
                }
            }
        }
        
        stage('Run Unit Tests') {
            when {
                expression { return params.RUN_UNIT.toBoolean() }
            }
            steps {
                script {
                    sh """
                       chmod u+x unit_tests.sh
                       ./unit_tests.sh
                       """
                }
            }
        }
        
        stage('Run Integration Tests') {
            when {
                expression { return params.RUN_INTEGRATION.toBoolean() }
            }
            steps {
                script {
                    sh """
                       chmod u+x integration_tests.sh
                       ./integration_tests.sh
                       """
                }
            }
        }
        
        stage('Application Launch Test') {
            steps {
                script {
                    sh "./${params.FILE_NAME}"
                }
            }
        }
        
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }
        
        stage('Deploy to Environment') {
            steps {
                script {
                    sshPublisher(
                        publishers: [
                            sshPublisherDesc(
                                configName: "ssh",
                                transfers: [
                                    sshTransfer(
                                        sourceFiles: "${params.FILE_NAME}",
                                        remoteDirectory: "${params.ENV}",
                                        execCommand: "echo \"Deployed ${params.FILE_NAME} to ${params.ENV} environment\""
                                    )
                                ],
                                usePromotionTimestamp: false,
                                useWorkspaceInPromotion: false,
                                verbose: true
                            )
                        ]
                    )
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed"
        }
        success {
            echo "Successfully deployed ${params.FILE_NAME} to ${params.ENV}"
        }
        failure {
            echo "Failed to deploy ${params.FILE_NAME} to ${params.ENV}"
        }
    }
}

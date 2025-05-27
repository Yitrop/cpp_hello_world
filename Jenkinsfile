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
        
        stage('Clean Workspace') {
            steps {
                deleteDir()
            }
        }
        
        stage('Print Info') {
            steps {
                script {
                    try {
                        sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)"'
                        sh 'echo "Hash: $(git rev-parse HEAD)"'
                        sh 'echo "g++ version: $(g++ --version)"'
                    } catch (Exception e) {
                        echo "WARNING: Some info commands failed"
                    }
                }
            }
        }
        
        stage('Build Executable file') {
            steps {
                script {
                    try {
                        sh "g++ app.cpp -o ${params.FILE_NAME}"
                    } catch (Exception e) {
                        error("Build failed: ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Run Unit Tests') {
            when {
                expression { return params.RUN_UNIT.toBoolean() }
            }
            steps {
                script {
                    try {
                        sh """
                           chmod u+x unit_tests.sh
                           ./unit_tests.sh
                           """
                    } catch (Exception e) {
                        error("Unit tests failed: ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Run Integration Tests') {
            when {
                expression { return params.RUN_INTEGRATION.toBoolean() }
            }
            steps {
                script {
                    try {
                        sh """
                           chmod u+x integration_tests.sh
                           ./integration_tests.sh
                           """
                    } catch (Exception e) {
                        error("Integration tests failed: ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Application Launch Test') {
            steps {
                script {
                    try {
                        sh "./${params.FILE_NAME}"
                    } catch (Exception e) {
                        error("Application test failed: ${e.getMessage()}")
                    }
                }
            }
        }
        
        stage('Deploy to Environment') {
            steps {
                script {
                    try {
                        sshPublisher(
                            publishers: [
                                sshPublisherDesc(
                                    configName: "ssh",
                                    transfers: [
                                        sshTransfer(
                                            sourceFiles: "${params.FILE_NAME}",
                                            remoteDirectory: "${params.ENV}",
                                            execCommand: """
                                                echo "Deployed ${params.FILE_NAME} to ${params.ENV} environment"
                                            """
                                        )
                                    ],
                                    usePromotionTimestamp: false,
                                    useWorkspaceInPromotion: false,
                                    verbose: true
                                )
                            ]
                        )
                    } catch (Exception e) {
                        error("Deployment failed: ${e.getMessage()}")
                    }
                }
            }
        }
    }
    
    post {
        always {
            echo "Pipeline execution completed"
            deleteDir()
        }
        success {
            echo "Successfully deployed ${params.FILE_NAME} to ${params.ENV}"
        }
        failure {
            echo "Failed to deploy ${params.FILE_NAME} to ${params.ENV}"
        }
    }
}

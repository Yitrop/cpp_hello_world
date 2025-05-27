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
            }
        }
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Print Info') {
            steps {
                sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)"'
                sh 'echo "Hash: $(git rev-parse HEAD)"'
                sh 'echo "g++ version: $(g++ --version)"'
            }
        }
        stage('Build Executable file') {
            steps {
                sh """g++ app.cpp -o ${params.FILE_NAME}"""
            }
        }
        stage('Run Unit Tests') {
            when {
                expression { return params.RUN_UNIT }
            }
            steps {
                sh """
                   chmod u+x unit_tests.sh
                   ./unit_tests.sh
                   """
            }
        }
        stage('Run Integration Tests') {
            when {
                expression { return params.RUN_INTEGRATION }
            }
            steps {
                sh """
                   chmod u+x integration_tests.sh
                   ./integration_tests.sh
                   """
            }
        }
        stage('Application Launch Test') {
            steps {
                sh """./${params.FILE_NAME}"""
            }
        }
        stage('Deploy to Environment') {
            steps {
                sshPublisher(
                    publishers: [
                        sshPublisherDesc(
                            configName: "ssh",
                            transfers: [
                                sshTransfer(
                                    sourceFiles: "${params.FILE_NAME}",
                                    remoteDirectory: "${params.ENV}"
                                )
                            ]
                        )
                    ]
                )
            }
        }
    }
    post {
        success {
            echo "Successfully deployed to ${params.ENV}"
        }
        failure {
            echo "Deployment to ${params.ENV} failed"
        }
    }
}

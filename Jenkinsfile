pipeline{
    agent any
    parameters{
        string(name: 'FILE_NAME', defaultValue: 'app', description: 'Имя исполняемого файла')
        booleanParam(name: 'RUN_UNIT', defaultValue: true, description: 'Запускать unit тесты')
        booleanParam(name: 'RUN_INTEGRATION', defaultValue: true, description: 'Запускать integration тесты')
    }
    stages{
        stage('Print Info'){
            steps{
                sh 'echo "Branch: $(git rev-parse --abbrev-ref HEAD)"'
                sh 'echo "Hash: $(git rev-parse HEAD)"'
                sh 'echo "g++ version: $(g++ --version)"'
            }
        }
        stage('Build Executable file'){
            steps{
                sh """g++ app.cpp -o ${params.FILE_NAME}"""
            }
        }
        stage('Run Unit Tests'){
            when {
                // Этап выполнится, если выражение true 
                expression { return params.RUN_UNIT }
            }
            steps{
                sh """
                   chmod u+x unit_tests.sh
                   ./unit_tests.sh
                   """
            }
        }
        stage('Run Integration Tests'){
            when {
                // Этап выполнится, если выражение true 
                expression { return params.RUN_INTEGRATION }
            }
            steps{
                sh """
                   chmod u+x integration_tests.sh
                   ./integration_tests.sh
                   """
            }
        }
        stage('Application Launch Test'){
            steps{
                // Запускаем исполняемый файл main из текущего каталога
                sh """./${params.FILE_NAME}"""
            }
        }
    }
	post{
		success{
			echo 'You can go home'
		}
		failure{
			echo 'Sit and work on'
		}
	}
}

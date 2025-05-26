pipeline{
    agent any
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
                // Компилируем main.cpp и сохраняем результат в файл main
                sh 'g++ main.cpp -o main'
            }
        }
        stage('Application Launch Test'){
            steps{
                // Запускаем исполняемый файл main из текущего каталога
                sh './main'
            }
        }
    }
}

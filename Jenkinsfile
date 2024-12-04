pipeline {
    agent any
    parameters {
        choice(name: 'BUILD_TYPE', choices: ['Scan Only', 'Scan + Deploy'], description: 'Select build type')
    }
    environment {
        SONAR_TOKEN = credentials('token_sonar') // Set this in Jenkins credentials
    }
    stages {
        stage('Install Dependencies') {
            steps {
                sh 'python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'pytest'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') { // Match Jenkins SonarQube server name
                    sh """
                    sonar-scanner \
                        -Dsonar.projectKey=vulpy \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    """
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' }
            }
            steps {
                echo 'Deploying to staging...'
                // Tambahkan script deployment di sini
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished!'
        }
    }
}

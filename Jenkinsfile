pipeline {
    agent any
    parameters {
        choice(name: 'BUILD_TYPE', choices: ['Scan Only', 'Scan + Deploy'], description: 'Select build type')
    }
    environment {
        SONAR_TOKEN = credentials('token_sonar')
    }
    stages {
        stage('Check Environment') {
            steps {
                sh '''
                if ! dpkg -s python3-venv > /dev/null 2>&1; then
                    echo "python3-venv is not installed. Installing now..."
                    sudo apt update && sudo apt install -y python3-venv
                fi
                '''
            }
        }
        stage('Install Dependencies') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        stage('Run Tests') {
            steps {
                sh '''
                echo "Direktori Sekarang: $(pwd)"
                ls -la
                . venv/bin/activate
                pytest test_example.py
            '''
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                    . venv/bin/activate
                    export PATH=/home/devsecops/sonar-scanner-4.8.0.2856-linux/bin:$PATH
                    sonar-scanner \
                        -Dsonar.projectKey=vulpy \
                        -Dsonar.sources=. \
                        -Dsonar.host.url=http://192.168.1.11:9000 \
                        -Dsonar.login=${SONAR_TOKEN}
                    '''
                }
            }
        }
        stage('Deploy to Staging') {
            when {
                expression { params.BUILD_TYPE == 'Scan + Deploy' }
            }
            steps {
                echo 'Deploying to staging...'
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished!'
        }
    }
}

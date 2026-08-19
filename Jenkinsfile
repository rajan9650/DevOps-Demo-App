pipeline {
    agent {
    label 'ubuntu-agent'
}

    environment {
        APP_NAME = 'devops-demo-app'
        CONTAINER_NAME = 'devops-demo-app'
        HOST_PORT = '8081'
        CONTAINER_PORT = '8080'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
stage('Install Required Tools') {
    steps {
        sh '''
            echo "================================"
            echo "Installing required tools on Agent"
            echo "================================"

            sudo apt update

            if ! command -v git >/dev/null 2>&1; then
                echo "Git not installed. Installing..."
                sudo apt install git -y
            else
                echo "Git already installed"
            fi

            if ! command -v mvn >/dev/null 2>&1; then
                echo "Maven not installed. Installing..."
                sudo apt install maven -y
            else
                echo "Maven already installed"
            fi

            if ! command -v docker >/dev/null 2>&1; then
                echo "Docker not installed. Installing..."
                sudo apt install docker.io -y
                sudo systemctl enable docker
                sudo systemctl start docker
            else
                echo "Docker already installed"
            fi
        '''
    }
}
        stage('Verify Tools') {
    steps {
        sh '''
            echo "===== JAVA ====="
            java -version

            echo "===== GIT ====="
            git --version

            echo "===== MAVEN ====="
            mvn -version

            echo "===== DOCKER ====="
            docker --version
        '''
    }
}

        stage('Test') {
            steps {
                sh 'mvn clean test'
            }
        }

        stage('Build JAR') {
            steps {
                sh 'mvn package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${APP_NAME}:${BUILD_NUMBER} .'
                sh 'docker tag ${APP_NAME}:${BUILD_NUMBER} ${APP_NAME}:latest'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true
                    docker run -d                       --name ${CONTAINER_NAME}                       --restart unless-stopped                       -p ${HOST_PORT}:${CONTAINER_PORT}                       ${APP_NAME}:latest
                '''
            }
        }

        stage('Health Check') {
            steps {
                sh '''
                    sleep 8
                    curl --fail http://localhost:${HOST_PORT}/health
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}

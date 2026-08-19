pipeline {
    agent any

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

        stage('Verify Tools') {
            steps {
                sh '''
                    java -version
                    mvn -version
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

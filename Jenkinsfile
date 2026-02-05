pipeline {
    agent any

    environment {
        APP_REPO = "https://github.com/Praj122/TodoSummaryAssistant.git"
        APP_BRANCH = "main"

        DOCKER_IMAGE = "ajithkumarreddy/todo-backend"
        DOCKER_TAG = "${BUILD_NUMBER}-${GIT_COMMIT}"
    }

    stages {

        stage('Checkout Application') {
            steps {
                deleteDir()
                git branch: "${APP_BRANCH}", url: "${APP_REPO}"
            }
        }

        stage('Build') {
            steps {
                dir('Backend/todo-summary-assistant') {
                    sh 'mvn -B clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                dir('Backend/todo-summary-assistant') {
                    sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .'
                }
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKER_IMAGE:$BUILD_NUMBER
                    '''
                }
            }
        }
    }
}

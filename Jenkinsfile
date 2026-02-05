pipeline {
    agent any

    environment {
        APP_REPO   = "https://github.com/Praj122/TodoSummaryAssistant.git"
        APP_BRANCH = "main"

        DOCKER_IMAGE = "ajithkumarreddy/todo-backend"
    }

    stages {

        stage('Checkout Application') {
            steps {
                deleteDir()
                git branch: "${APP_BRANCH}", url: "${APP_REPO}"
            }
        }

        stage('Generate Image Tag') {
            steps {
                script {
                    COMMIT = sh(
                        script: "git rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()

                    env.IMAGE_TAG = "${BUILD_NUMBER}-${COMMIT}"
                }
            }
        }

        stage('Build Jar') {
            steps {
                dir('Backend/todo-summary-assistant') {
                    sh 'mvn -B clean package -DskipTests'
                }
            }
        }

        stage('Prepare Docker Context') {
            steps {
                sh '''
                cp Dockerfile Backend/todo-summary-assistant/
                cp .dockerignore Backend/todo-summary-assistant/ 2>/dev/null || true
                '''
            }
        }

        stage('Docker Build') {
            steps {
                dir('Backend/todo-summary-assistant') {
                    sh 'docker build -t $DOCKER_IMAGE:$IMAGE_TAG .'
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
                    docker push $DOCKER_IMAGE:$IMAGE_TAG
                    docker logout
                    '''
                }
            }
        }
    }
}

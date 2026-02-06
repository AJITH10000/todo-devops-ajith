pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ajithkumarreddy/todo-backend"
    }

    stages {

        stage('Build Docker Image') {
            steps {
                sh '''
                docker build -t $DOCKER_IMAGE:$BUILD_NUMBER .
                docker tag $DOCKER_IMAGE:$BUILD_NUMBER $DOCKER_IMAGE:latest
                '''
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
                    docker push $DOCKER_IMAGE:latest

                    docker logout
                    '''
                }
            }
        }

        stage('Cleanup Docker (VERY IMPORTANT)') {
            steps {
                sh '''
                docker container prune -f
                docker image prune -af
                docker builder prune -af
                '''
            }
        }
    }
}

pipeline {
    agent any

    environment {
        IMAGE_NAME = "ajithkumarreddy/todo-backend"
        IMAGE_TAG = "${env.GIT_COMMIT}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                docker build -t $IMAGE_NAME:$IMAGE_TAG .
                '''
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh '''
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $IMAGE_NAME:$IMAGE_TAG
                    docker logout
                    '''
                }
            }
        }

        stage('Update GitOps Repo') {
            steps {
                sh '''
                git clone https://github.com/<your-gitops-repo>.git
                cd <your-gitops-repo>

                sed -i "s|image:.*|image: $IMAGE_NAME:$IMAGE_TAG|" k8s/deployment.yaml

                git config user.email "jenkins@ci"
                git config user.name "jenkins"
                git add .
                git commit -m "Update image to $IMAGE_TAG"
                git push
                '''
            }
        }
    }
}

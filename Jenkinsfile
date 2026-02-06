pipeline {
    agent any

    environment {
        IMAGE_NAME = "ajithkumarreddy/todo-backend"
        IMAGE_TAG = "${env.GIT_COMMIT}"
        GITOPS_REPO = "github.com/AJITH10000/todo-gitops.git"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend') {
            steps {
                sh '''
                cd Backend/todo-summary-assistant
                mvn -B clean package -DskipTests
                '''
            }
        }

        stage('Run Tests') {
            steps {
                sh '''
                cd Backend/todo-summary-assistant
                mvn test
                '''
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
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN'
                )]) {
                    sh '''
                    rm -rf gitops
                    git clone https://$GIT_USER:$GIT_TOKEN@$GITOPS_REPO gitops

                    cd gitops/apps/todo

                    sed -i "s|image: .*todo-backend.*|image: $IMAGE_NAME:$IMAGE_TAG|" backend-deployment.yaml

                    git config user.email "jenkins@ci"
                    git config user.name "jenkins"

                    git add .
                    git commit -m "Update backend image to $IMAGE_TAG"
                    git push
                    '''
                }
            }
        }
    }
}

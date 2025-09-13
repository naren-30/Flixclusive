pipeline {
    agent any

    environment {
        DOCKER_HUB_USER = 'naren3005'
        IMAGE_NAME = 'netflix'
        IMAGE_TAG = 'v1'
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naren-30/Flixclusive.git'
            }
        }

        stage('Build with Maven (via Docker)') {
            steps {
                sh '''
                    docker run --rm -v "$PWD":/app -w /app maven:3.8.7-openjdk-17 mvn clean package
                '''
            }
        }

        stage('Docker Build Image') {
            steps {
                sh 'docker build -t $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG .'
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG'
                }
            }
        }

        stage('Docker Swarm Deploy') {
            steps {
                sh '''
                    docker swarm init || true
                    docker service rm hotserv || true
                    docker service create \
                        --name hotserv \
                        --publish published=8009,target=8080 \
                        --replicas 3 \
                        $DOCKER_HUB_USER/$IMAGE_NAME:$IMAGE_TAG
                '''
            }
        }
    }
}

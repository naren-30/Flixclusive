pipeline {
    agent any

    tools {
        maven 'Maven 3.8.7'   // Use the name you configured in Jenkins
    }

    environment {
        DOCKER_HUB_USER = 'naren3005'     // Your Docker Hub username
        IMAGE_NAME = 'netflix'            // Your Docker image name
        IMAGE_TAG = 'v1'                  // Your Docker image tag
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/naren-30/Flixclusive.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
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
                    credentialsId: 'docker', // Replace with your Jenkins credential ID
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

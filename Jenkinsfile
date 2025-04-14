pipeline {
    agent {
        docker {
            image 'maven:3.9.3-eclipse-temurin-17'
            args '-v /root/.m2:/root/.m2'
        }
    }

    environment {
        DOCKER_IMAGE = 'pipilikaka/springboot-hello-world'
        VERSION = "v1.${BUILD_NUMBER}"
        CONTAINER_NAME = "hello-world-app"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE:$VERSION .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-token', variable: 'DOCKERHUB_TOKEN')]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login -u pipilikaka --password-stdin
                        docker push $DOCKER_IMAGE:$VERSION
                    '''
                }
            }
        }

        stage('Run Container') {
            steps {
                sh '''
                    docker stop $CONTAINER_NAME || true
                    docker rm $CONTAINER_NAME || true
                    docker run -d --name $CONTAINER_NAME -p 8081:8080 $DOCKER_IMAGE:$VERSION
                '''
            }
        }

        stage('Cleanup') {
            steps {
                sh 'docker rmi $DOCKER_IMAGE:$VERSION || true'
            }
        }
    }

    post {
        success {
            echo "Deployed and running at http://<JENKINS-HOST>:8081"
        }
        failure {
            echo "Pipeline failed during one of the steps."
        }
    }
}

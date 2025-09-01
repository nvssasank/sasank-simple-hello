pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "sri642/springboot-app-hello:${BUILD_NUMBER}"
        DOCKER_HUB_CREDS = "Docker-sasank"
        KUBE_NAMESPACE = "sasank"
    }
    tools{
        maven 'maven3'
    }
    stages {
        stage('Checkout') {
            steps {
                git credentialsId: 'Github_token', url: 'https://github.com/nvssasank/sasank-simple-hello.git', branch: 'main'
            }
        }
        stage('Build with Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: "$DOCKER_HUB_CREDS", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_IMAGE'
                }
            }
        }
        stage('Deploy to EKS') {
            steps {
                sh '''
                  sed -i "s@<IMAGE_PLACEHOLDER>@$DOCKER_IMAGE@g" k8s/deployment.yaml
                  kubectl apply -f k8s/namespace.yaml
                  kubectl apply -n srilatha-namespace -f k8s/deployment.yaml
                '''
            }
        }
    }
}

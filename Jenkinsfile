pipeline {
    agent any

    environment {
        DOCKER_IMAGE   = "sasank1219/springboot-app"
        DOCKER_TAG     = "latest"
        KUBE_NAMESPACE = "sasank"
    }

    tools {
        maven "maven3"  // Configure this in Jenkins -> Manage Jenkins -> Tools -> Maven
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/nvssasank/sasank-simple-hello.git'
            }
        }

        stage('Build with Maven') {
            steps {
                sh "mvn clean package -DskipTests"
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKER_IMAGE:$DOCKER_TAG ."
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push $DOCKER_IMAGE:$DOCKER_TAG"
                }
            }
        }

        stage('Deploy to EKS') {
            steps {
                script {
                    // Assumes you already have kubeconfig set up on Jenkins agent
                    sh "kubectl apply -n $KUBE_NAMESPACE -f k8s/deployment.yaml"
                    sh "kubectl apply -n $KUBE_NAMESPACE -f k8s/service.yaml"
                }
            }
        }
    }
}

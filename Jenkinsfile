pipeline {
    agent any 

    environment {
        DOCKER_CREDENTIALS_ID = 'roseaw-dockerhub'  
        DOCKER_IMAGE = 'cithit/sapkotp2'
        IMAGE_TAG = "build-${BUILD_NUMBER}"
        GITHUB_URL = 'https://github.com/parbatisapkota/225-lab3-9.git'
        KUBECONFIG = credentials('sapkotp2-225-sp26')
    }

    stages {

        stage('Checkout') {
            steps {
                cleanWs()
                checkout([$class: 'GitSCM', branches: [[name: '*/main']],
                          userRemoteConfigs: [[url: "${GITHUB_URL}"]]])
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'python3 -m pip install --upgrade pip'
                sh 'pip install -r requirements.txt'
            }
        }

        stage('Run Tests (pytest)') {
            steps {
                sh 'pytest'
            }
        }

        stage('Security Scan (Bandit)') {
            steps {
                sh 'pip install bandit'
                sh 'bandit -r .'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry-1.docker.io', 'roseaw-dockerhub') {
                        docker.build("${DOCKER_IMAGE}:${IMAGE_TAG}")
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_CREDENTIALS_ID}") {
                        docker.image("${DOCKER_IMAGE}:${IMAGE_TAG}").push()
                    }
                }
            }
        }

        stage('Deploy to Dev Environment') {
            steps {
                script {
                    def kubeConfig = readFile(KUBECONFIG)
                    sh "sed -i 's|${DOCKER_IMAGE}:latest|${DOCKER_IMAGE}:${IMAGE_TAG}|' deployment-dev.yaml"
                    sh "kubectl apply -f deployment-dev.yaml"
                }
            }
        }
        
        stage('Check Kubernetes Cluster') {
            steps {
                script {
                    sh "kubectl get all"
                }
            }
        }
    }

    post {
        success {
            slackSend color: "good", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        unstable {
            slackSend color: "warning", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
        failure {
            slackSend color: "danger", message: "Build Completed: ${env.JOB_NAME} ${env.BUILD_NUMBER}"
        }
    }
}
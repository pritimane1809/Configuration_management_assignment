pipeline {
    agent any

    environment {
        IMAGE_NAME = "bluegreen-app"
        VERSION = "v1"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main', url: 'https://github.com/Shreyash1928/blue-green-app'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${VERSION} ."
            }
        }

        stage('Blue-Green Deployment') {
            steps {
                script {
                    echo "Checking if BLUE container is running..."
                    def BLUE = sh(script: "docker ps --filter name=blue-app -q", returnStdout: true).trim()

                    if (BLUE) {
                        // BLUE is running → Deploy GREEN
                        echo "BLUE is running → Deploying GREEN on port 8082"
                        sh """
                            docker rm -f green-app || true
                            docker run -d \
                                -p 8082:8080 \
                                --name green-app \
                                -e DEPLOY_COLOR=green \
                                ${IMAGE_NAME}:${VERSION}
                        """
                    } else {
                        // BLUE NOT running → Deploy BLUE
                        echo "BLUE is NOT running → Deploying BLUE on port 8081"
                        sh """
                            docker rm -f blue-app || true
                            docker run -d \
                                -p 8081:8080 \
                                --name blue-app \
                                -e DEPLOY_COLOR=blue \
                                ${IMAGE_NAME}:${VERSION}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "Blue-Green Deployment Successful!"
            echo "BLUE  → http://localhost:8081"
            echo "GREEN → http://localhost:8082"
        }
    }
}

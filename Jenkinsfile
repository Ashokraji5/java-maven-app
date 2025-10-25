pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ashokraji/tomcat"
        DOCKER_TAG = "9.0-${BUILD_NUMBER}" // You can also use Git commit hash: "9.0-${GIT_COMMIT.take(7)}"
    }

    tools {
        maven 'maven'  // Ensure the Maven tool is configured in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Checking out the code..."
                checkout scm  // Simplifies Git checkout process
            }
        }

        stage('Build with Maven') {
            steps {
                echo "🔧 Building the project with Maven..."
                script {
                    try {
                        sh 'mvn clean package -DskipTests -e -X'
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e  // Propagate error
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                script {
                    try {
                        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e  // Propagate error
                    }
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                echo "📤 Pushing Docker image to DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',  // Make sure credentialsId is correct
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        try {
                            // Login to DockerHub and push the image
                            sh """
                                echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                            """
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            throw e  // Propagate error
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded — Docker image pushed to DockerHub!'
        }
        failure {
            echo '❌ Pipeline failed — Check the logs for details!'
        }
        always {
            // Clean up or additional notifications can be added here
            echo 'Cleaning up resources...'
        }
    }
}

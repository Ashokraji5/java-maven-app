pipeline {
    agent any  // Run on any available agent (can be restricted to master if you prefer)

    environment {
        DOCKER_IMAGE = "ashokraji/tomcat"
        DOCKER_TAG = "9.0-${BUILD_NUMBER}"
    }

    tools {
        maven 'maven'  // Assuming Maven is already configured in Jenkins
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Cloning the repository from GitHub..."
                git url: 'git@github.com:Ashokraji5/java-maven-app.git', branch: 'main', credentialsId: 'github-ssh-credentials'
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
                        error "Maven build failed!"
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
                        error "Docker build failed!"
                    }
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                echo "📤 Pushing Docker image to DockerHub..."
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    script {
                        try {
                            sh """
                                echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin
                                docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                            """
                        } catch (Exception e) {
                            currentBuild.result = 'FAILURE'
                            error "Docker push failed!"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded — Image pushed to DockerHub!'
        }
        failure {
            echo '❌ Pipeline failed — Check the logs!'
        }
    }
}

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = "ashokraji/tomcat"
        DOCKER_TAG = "9.0-${BUILD_NUMBER}"
        SONARQUBE_URL = 'http://52.90.161.136:9000'
        SONARQUBE_TOKEN = credentials('sonarqube-token')
    }

    tools {
        maven 'maven'
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Ashokraji5/java-maven-app.git',
                    credentialsId: 'github-credentials'
            }
        }

        stage('Build with Maven') {
            steps {
                echo "🔧 Building the project with Maven..."
                sh 'mvn clean package -DskipTests -e -X'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "🔍 Running SonarQube analysis..."
                sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=my-project-key \
                    -Dsonar.host.url=${SONARQUBE_URL} \
                    -Dsonar.login=${SONARQUBE_TOKEN}
                """
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
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
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
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


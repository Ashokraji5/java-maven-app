pipeline {
    agent any

    environment {
        ARTIFACTORY_REPO = "libs-release-local"  // Change to your actual repository (libs-release-local or libs-snapshot-local)
        ARTIFACTORY_URL = "https://<your-artifactory-ip>:8081/artifactory"  // Your Artifactory URL
        ARTIFACT_NAME = "my-artifact"  // Replace with your artifact name
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

        stage('Deploy to Artifactory') {
            steps {
                echo "📦 Deploying artifact to Artifactory..."
                script {
                    try {
                        // Replace the artifact details as per your project
                        sh """
                            mvn deploy:deploy-file \
                                -DgroupId=com.example \
                                -DartifactId=${ARTIFACT_NAME} \
                                -Dversion=${BUILD_NUMBER} \
                                -Dpackaging=jar \
                                -Dfile=target/${ARTIFACT_NAME}-${BUILD_NUMBER}.jar \
                                -DrepositoryId=artifactory \
                                -Durl=${ARTIFACTORY_URL}/${ARTIFACTORY_REPO}
                        """
                    } catch (Exception e) {
                        currentBuild.result = 'FAILURE'
                        throw e  // Propagate error
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Pipeline succeeded — Artifact deployed to Artifactory!'
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

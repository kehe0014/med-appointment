pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/appointment-app"
        IMAGE_TAG = "latest"
        PACKAGES_URL = "https://maven.pkg.github.com/${GITHUB_OWNER}/${GITHUB_REPO}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify GitHub Authentication') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔐 Validating GitHub authentication..."
                        def authStatus = sh(
                            script: '''curl -s -o /dev/null -w "%{http_code}" \
                                -H "Authorization: token $GITHUB_TOKEN" \
                                -H "Accept: application/vnd.github.v3+json" \
                                https://api.github.com/user''',
                            returnStdout: true
                        ).trim()
                        if (authStatus != "200") {
                            error("❌ GitHub authentication failed (HTTP ${authStatus})")
                        }
                        echo "✅ GitHub authentication successful"
                    }
                }
            }
        }

        stage('Build Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Build docker image with the jar produced by Maven
                        sh """
                        docker build -f docker/Dockerfile \
                          --build-arg JAR_FILE=target/Appointment-0.0.1-SNAPSHOT.jar \
                          -t ${IMAGE_NAME}:${IMAGE_TAG} .
                        """
                    }
                }
            }
        }

        stage('Push Docker Image to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Authenticate Docker to GHCR
                        sh "echo $GITHUB_TOKEN | docker login ghcr.io -u ${GITHUB_OWNER} --password-stdin"

                        // Push image
                        sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"

                        // Optional: logout for cleanup
                        sh "docker logout ghcr.io"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline succeeded: image pushed to ${IMAGE_NAME}:${IMAGE_TAG}"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
        always {
            // Cleanup Docker images to free space on agent
            sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true"
        }
    }
}

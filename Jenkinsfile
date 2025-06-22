pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/${GITHUB_REPO}"  // Lowercase for GHCR
        VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim().toLowerCase()
        TIMESTAMP = new Date().format('yyyyMMdd-HHmmss')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                // Generate Maven wrapper if missing
                sh 'mvn -N wrapper:wrapper || true'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
                script {
                    def jarFile = "target/${GITHUB_REPO}-${env.VERSION}.jar"
                    if (!fileExists(jarFile)) {
                        error("❌ JAR file not found: ${jarFile}")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -f docker/Dockerfile \
                      --build-arg JAR_FILE=target/${GITHUB_REPO}-${VERSION}.jar \
                      -t ${IMAGE_NAME}:${VERSION} \
                      -t ${IMAGE_NAME}:latest \
                      -t ${IMAGE_NAME}:${TIMESTAMP} .
                    """
                }
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN']) {
                    script {
                        // Authenticate with GHCR
                        sh """
                        echo \$GITHUB_TOKEN | docker login ghcr.io -u ${GITHUB_OWNER} --password-stdin || exit 1
                        """
                        
                        // Push with retry logic
                        retry(3) {
                            sh """
                            docker push ${IMAGE_NAME}:${VERSION}
                            docker push ${IMAGE_NAME}:latest
                            docker push ${IMAGE_NAME}:${TIMESTAMP}
                            """
                        }
                        
                        sh "docker logout ghcr.io"
                    }
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                echo "🚀 Deployment would execute here (kubectl/helm commands)"
                // sh "kubectl set image deployment/${GITHUB_REPO} ${GITHUB_REPO}=${IMAGE_NAME}:${VERSION}"
            }
        }
    }

    post {
        always {
            script {
                // Cleanup images
                sh """
                docker rmi ${IMAGE_NAME}:${VERSION} || true
                docker rmi ${IMAGE_NAME}:latest || true
                docker rmi ${IMAGE_NAME}:${TIMESTAMP} || true
                """
            }
        }
        success {
            echo "🎉 Pipeline succeeded! Image: ${IMAGE_NAME}:${VERSION}"
            slackSend(color: 'good', message: "Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
            slackSend(color: 'danger', message: "Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}")
        }
    }
}
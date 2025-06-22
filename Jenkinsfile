pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        // GHCR requires lowercase package names
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/${GITHUB_REPO.toLowerCase()}"
        VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
        // Convert version to lowercase and replace special characters
        SAFE_VERSION = "${VERSION}".toLowerCase().replace('-SNAPSHOT', '-snapshot')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                // Ensure Maven wrapper exists
                sh 'mvn -N wrapper:wrapper || true'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
                script {
                    def jarFile = "target/${GITHUB_REPO}-${VERSION}.jar"
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
                      -t ${IMAGE_NAME}:${SAFE_VERSION} \
                      -t ${IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GH_TOKEN']) {
                    script {
                        // First authenticate - CRITICAL STEP
                        sh """
                        echo "Authenticating with GHCR..."
                        echo \$GH_TOKEN | docker login ghcr.io \
                          -u ${GITHUB_OWNER} \
                          --password-stdin || exit 1
                        """
                        
                        // Push with retry logic
                        retry(3) {
                            sh """
                            echo "Pushing ${IMAGE_NAME}:${SAFE_VERSION}"
                            docker push ${IMAGE_NAME}:${SAFE_VERSION}
                            
                            echo "Pushing ${IMAGE_NAME}:latest"
                            docker push ${IMAGE_NAME}:latest
                            """
                        }
                        
                        // Cleanup authentication
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
                echo "🚀 Deployment would execute here"
                // Example:
                // sh "kubectl set image deployment/${GITHUB_REPO} ${GITHUB_REPO}=${IMAGE_NAME}:${SAFE_VERSION}"
            }
        }
    }

    post {
        always {
            script {
                // Cleanup images
                sh """
                docker rmi ${IMAGE_NAME}:${SAFE_VERSION} || true
                docker rmi ${IMAGE_NAME}:latest || true
                """
            }
        }
        success {
            echo "🎉 Success! Image pushed to:"
            echo " - ${IMAGE_NAME}:${SAFE_VERSION}"
            echo " - ${IMAGE_NAME}:latest"
        }
        failure {
            echo "❌ Pipeline failed - check logs"
        }
    }
}
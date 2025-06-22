pipeline {
        agent {
        label 'tdk-desk-agent-01' // Use my dedicated worker from my local network
    }

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/${GITHUB_REPO.toLowerCase()}"
        // Get exact JAR filename from Maven build
        JAR_FILE = sh(script: 'find target -name "*.jar" -printf "%f\n" | head -1', returnStdout: true).trim()
        VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
        SAFE_VERSION = "${VERSION}".toLowerCase().replace('-SNAPSHOT', '-snapshot')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                sh 'mvn -N wrapper:wrapper || true'
            }
        }

        stage('Build') {
            steps {
                sh './mvnw clean package -DskipTests'
                script {
                    if (!fileExists("target/${JAR_FILE}")) {
                        error("❌ JAR file not found: target/${JAR_FILE}")
                    }
                    echo "✅ Found JAR file: target/${JAR_FILE}"
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                    docker build -f docker/Dockerfile \
                      --build-arg JAR_FILE=target/${JAR_FILE} \
                      -t ${IMAGE_NAME}:${SAFE_VERSION} \
                      -t ${IMAGE_NAME}:latest .
                    """
                }
            }
        }

        stage('Push to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GH_TOKEN')]) {
                    script {
                        sh """
                        echo \$GH_TOKEN | docker login ghcr.io \
                          -u ${GITHUB_OWNER} \
                          --password-stdin || exit 1
                        """
                        
                        retry(3) {
                            sh """
                            docker push ${IMAGE_NAME}:${SAFE_VERSION}
                            docker push ${IMAGE_NAME}:latest
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
                echo "🚀 Deployment would execute here"
            }
        }
    }

    post {
        always {
            script {
                sh """
                docker rmi ${IMAGE_NAME}:${SAFE_VERSION} || true
                docker rmi ${IMAGE_NAME}:latest || true
                """
            }
        }
        success {
            echo "🎉 Success! Image pushed to GHCR"
            echo "- ${IMAGE_NAME}:${SAFE_VERSION}"
            echo "- ${IMAGE_NAME}:latest"
        }
        failure {
            echo "❌ Pipeline failed - check logs"
        }
    }
}
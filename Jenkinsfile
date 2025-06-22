pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/appointment-app"
        PACKAGES_URL = "https://maven.pkg.github.com/${GITHUB_OWNER}/${GITHUB_REPO}"
        // Get version from pom.xml or use timestamp
        VERSION = sh(script: 'mvn help:evaluate -Dexpression=project.version -q -DforceStdout', returnStdout: true).trim()
        TIMESTAMP = new Date().format('yyyyMMdd-HHmmss')
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
                script {
                    // Verify built JAR
                    def jarFile = "target/med-rdv-${VERSION}.jar"
                    if (!fileExists(jarFile)) {
                        error("❌ JAR file not found: ${jarFile}")
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${IMAGE_NAME}:${VERSION}", 
                               "--build-arg JAR_FILE=target/med-rdv-${VERSION}.jar -f docker/Dockerfile .")
                    docker.image("${IMAGE_NAME}:${VERSION}").push()
                    
                    // Also tag as latest
                    sh "docker tag ${IMAGE_NAME}:${VERSION} ${IMAGE_NAME}:latest"
                    docker.image("${IMAGE_NAME}:latest").push()
                    
                    // Add timestamp tag for traceability
                    sh "docker tag ${IMAGE_NAME}:${VERSION} ${IMAGE_NAME}:${TIMESTAMP}"
                    docker.image("${IMAGE_NAME}:${TIMESTAMP}").push()
                }
            }
        }

        stage('Deploy') {
            when {
                branch 'main' // Only deploy from main branch
            }
            steps {
                script {
                    
                    sh """
                    kubectl set image deployment/appointment-app \
                    appointment-app=${IMAGE_NAME}:${VERSION} --record
                    """
                }
            }
        }
    }

    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ Success: ${env.JOB_NAME} #${env.BUILD_NUMBER}\n" +
                        "Version: ${VERSION}\n" +
                        "Image: ${IMAGE_NAME}:${VERSION}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}\n" +
                        "See: ${env.BUILD_URL}"
            )
        }
        always {
            // Cleanup Docker images
            script {
                sh "docker rmi ${IMAGE_NAME}:${VERSION} || true"
                sh "docker rmi ${IMAGE_NAME}:latest || true"
                sh "docker rmi ${IMAGE_NAME}:${TIMESTAMP} || true"
            }
        }
    }
}
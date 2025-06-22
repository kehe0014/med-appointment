pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${env.GITHUB_OWNER}/appointment-app:latest"
        PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
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
                            script: 'curl -s -o /dev/null -w "%{http_code}" ' +
                                    '-H "Authorization: token $GITHUB_TOKEN" ' +
                                    '-H "Accept: application/vnd.github.v3+json" ' +
                                    'https://api.github.com/user',
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

        stage('Build JAR') {
            steps {
                sh 'mvn clean package -DskipTests'
                sh 'ls -la target/' //Debugging step to list files in target directory
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -f docker/Dockerfile -t ${IMAGE_NAME} ."
                }
            }
        }

        stage('Push Docker Image to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        sh "echo $GITHUB_TOKEN | docker login ghcr.io -u ${GITHUB_OWNER} --password-stdin"
                        sh "docker push ${IMAGE_NAME}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Pipeline completed: Docker image pushed to GitHub Container Registry"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for errors."
        }
    }
}

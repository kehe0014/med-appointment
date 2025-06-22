pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        IMAGE_NAME = "ghcr.io/${GITHUB_OWNER}/appointment-app:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('GitHub Auth Test') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        echo "🔐 Test GitHub token..."
                        curl -s -o /dev/null -w "%{http_code}" \
                        -H "Authorization: token $GITHUB_TOKEN" \
                        https://api.github.com/user | grep 200 || exit 1
                    '''
                }
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker compose -f docker-compose.yml build appointment-app'
            }
        }

        stage('Tag and Push Image to GHCR') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        echo "${GITHUB_TOKEN}" | docker login ghcr.io -u ${GITHUB_OWNER} --password-stdin

                        # Tag image (si `docker-compose build` produit `appointment-app:latest`)
                        docker tag appointment-app:latest ${IMAGE_NAME}

                        # Push vers GitHub Container Registry
                        docker push ${IMAGE_NAME}
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleanup done"
        }
        success {
            echo "✅ Pipeline OK: image poussée sur GHCR"
        }
        failure {
            echo "❌ Échec - vérifie les étapes ci-dessus"
        }
    }
}

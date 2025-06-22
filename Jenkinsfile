pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
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
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=github::default::${env.PACKAGES_URL} \
                        -DrepositoryId=github \
                        -Durl=${env.PACKAGES_URL} \
                        -Dtoken=\$GITHUB_TOKEN
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Success! Artifacts deployed to GitHub Packages"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
    }
}

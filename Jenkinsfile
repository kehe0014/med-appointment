pipeline {
    agent { label 'tdk-desk-agent-01' }

    environment {
        GITHUB_USER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        GITHUB_TOKEN = credentials('GITHUB_ACESS_TOKEN') // Jenkins Credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/${env.GITHUB_USER}/${env.GITHUB_REPO}.git", branch: 'main'
            }
        }

    
        stage('Vérifier l’accès à GitHub Packages') {
          steps {
            withCredentials([
              string(credentialsId: 'GITHUB_ACESS_TOKEN', variable: 'ACCESS_TOKEN')
            ]) {
              echo '🔐 Vérification de la connexion à GitHub Packages...'
    
              // Simuler une requête HEAD vers le Maven package registry GitHub
              sh '''
                curl -I -u $GITHUB_OWNER:$ACCESS_TOKEN \
                  https://maven.pkg.github.com/$GITHUB_OWNER/$GITHUB_REPO/
              '''
            }
          }
        }
  // -----------------------------------
    }

    post {
        success {
            echo '✅ Build & Deployment successful.'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}

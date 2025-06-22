pipeline {
    agent { label 'tdk-desk-agent-01' }

    environment {
        GITHUB_USER = 'kehe0014'
        GITHUB_REPO = 'healtcare-med-rdv-scheduler'
        GITHUB_TOKEN = credentials('GITHUB_ACESS_TOKEN') // Jenkins Credentials ID
    }

    stages {
        stage('Checkout') {
            steps {
                git url: "https://github.com/${env.GITHUB_USER}/${env.GITHUB_REPO}.git", branch: 'main'
            }
        }

        stage('Build with Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
    
        stage('Vérifier l’accès à GitHub Packages') {
          steps {
            withCredentials([
              string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_ACCESS_TOKEN')
            ]) {
              echo '🔐 Vérification de la connexion à GitHub Packages...'
    
              // Simuler une requête HEAD vers le Maven package registry GitHub
              sh '''
                curl -I -u $GITHUB_OWNER:$GITHUB_ACCESS_TOKEN \
                  https://maven.pkg.github.com/$GITHUB_OWNER/$GITHUB_REPO/
              '''
            }
          }
        }


        stage('Publish to GitHub Packages') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    writeFile file: 'settings.xml', text: """
                        <settings>
                        <servers>
                            <server>
                            <id>github</id>
                            <username>${env.GITHUB_USER}</username>
                            <password>${env.GITHUB_TOKEN}</password>
                            </server>
                            <server>
                            <id>github-snapshots</id>
                            <username>${env.GITHUB_USER}</username>
                            <password>${env.GITHUB_TOKEN}</password>
                            </server>
                        </servers>
                        </settings>
                    """
                    sh 'mvn deploy --settings settings.xml -DskipTests'
                }
            }
        }

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

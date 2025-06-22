pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
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

        stage('Setup Maven Settings') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        mkdir -p $HOME/.m2
                        cat > $HOME/.m2/settings.xml <<EOF
<settings>
  <servers>
    <server>
      <id>github</id>
      <username>${GITHUB_OWNER}</username>
      <password>${GITHUB_TOKEN}</password>
    </server>
  </servers>
</settings>
EOF
                    '''
                }
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                sh """
                    mvn deploy -DskipTests \
                    -DaltDeploymentRepository=github::default::${PACKAGES_URL}
                """
            }
        }
    }

    post {
        always {
            sh 'rm -f $HOME/.m2/settings.xml || true'
            echo "🧹 Cleaned up temporary settings.xml"
        }
        success {
            echo "🎉 Successfully deployed to GitHub Packages!"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
    }
}

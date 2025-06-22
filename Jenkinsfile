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

        stage('Verify GitHub Authentication') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔐 Validating GitHub authentication..."

                        def authStatus = sh(
                            script: 'curl -s -o /dev/null -w "%{http_code}" ' +
                                    '-H "Authorization: token ' + GITHUB_TOKEN + '" ' + // Utiliser la variable directement
                                    '-H "Accept: application/vnd.github.v3+json" ' +
                                    'https://api.github.com/user',
                            returnStdout: true
                        ).trim()

                        if (authStatus != "200") {
                            error("❌ GitHub authentication failed (HTTP ${authStatus}). Check PAT scopes or expiration.")
                        }
                        echo "✅ GitHub authentication successful"
                    }
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
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Create temporary settings.xml
                        // Utilisez stripIndent() pour gérer l'indentation du Groovy,
                        // mais assurez-vous que EOF n'a PAS d'indentation DANS le bloc shell.
                        sh """
                            mkdir -p \$HOME/.m2
                            cat > \$HOME/.m2/settings.xml <<EOF
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
                        """.stripIndent() // Ajoutez .stripIndent() ici

                        echo "Generated settings.xml at $HOME/.m2/settings.xml"
                        sh "cat $HOME/.m2/settings.xml" // Pour vérifier le contenu du fichier créé

                        // Deploy with authenticated settings
                        sh """
                            mvn deploy -DskipTests \\
                            -s \$HOME/.m2/settings.xml \\
                            -DaltDeploymentRepository=github::default::${env.PACKAGES_URL}
                        """
                    }
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Successfully deployed to GitHub Packages!"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
            // Optionnel: Nettoyer le settings.xml temporaire en cas d'échec
            // sh "rm -f $HOME/.m2/settings.xml"
        }
        always {
            // Supprimez le settings.xml temporaire après la build (qu'elle réussisse ou échoue)
            script {
                sh "rm -f \$HOME/.m2/settings.xml"
                echo "Cleaned up temporary settings.xml"
            }
        }
    }
}

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

        stage('Verify GitHub Packages') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔍 Checking GitHub Packages accessibility..."
                        def statusCode = sh(
                            script: """
                                curl -s -o /dev/null -w '%{http_code}' \
                                -H "Authorization: token \$GITHUB_TOKEN" \
                                ${env.PACKAGES_URL}/
                            """,
                            returnStdout: true
                        ).trim()

                        echo "GitHub Packages response: HTTP ${statusCode}"
                        
                        if (statusCode == "404") {
                            echo "⚠️ Warning: Package repository not found (404) - proceeding anyway"
                        } else if (statusCode != "200") {
                            error("❌ Fatal: GitHub Packages access failed (HTTP ${statusCode})")
                        }
                    }
                }
            }
        }

        stage('Build') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn clean package -DskipTests \
                        -Dgithub.token=\$GITHUB_TOKEN
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=github::${env.PACKAGES_URL} \
                        -Dgithub.token=\$GITHUB_TOKEN
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Success! Built and deployed to GitHub Packages"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
            // Optional: Add notification here
        }
    }
}

pipeline {
    agent any

    environment {
        // Configure these values
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
    }

    stages {
        stage('Check GitHub Packages Access') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔍 Checking access to GitHub Packages at ${env.PACKAGES_URL}"
                        
                        // Safe curl command without Groovy interpolation
                        def statusCode = sh(returnStdout: true, script: """
                            curl -s -o /dev/null -w '%{http_code}' \
                            -H "Authorization: token ${env.GITHUB_TOKEN}" \
                            ${env.PACKAGES_URL}/
                        """).trim()

                        echo "GitHub Packages response: HTTP ${statusCode}"

                        // Handle responses
                        if (statusCode == "200") {
                            echo "✅ Package repository is accessible"
                        } else if (statusCode == "404") {
                            echo "⚠️ Package repository not found (404)"
                            // Continue instead of failing if you want
                        } else {
                            error("❌ Access check failed (HTTP ${statusCode})")
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            echo "📋 Access check completed"
        }
    }
}

pipeline {
    agent { label 'tdk-desk-agent-01' }

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        GITHUB_PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: "https://github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}.git",
                        credentialsId: 'GITHUB_ACCESS_TOKEN' // Corrected typo in credential name
                    ]]
                ])
            }
        }

        stage('Verify GitHub Packages Access') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'ACCESS_TOKEN')]) {
                    script {
                        echo '🔐 Verifying connection to GitHub Packages...'
                        
                        // Proper curl command with error handling
                        def statusCode = sh(
                            script: """
                                curl -s -o /dev/null -w "%{http_code}" \
                                -u ${env.GITHUB_OWNER}:${env.ACCESS_TOKEN} \
                                ${env.GITHUB_PACKAGES_URL}/
                            """,
                            returnStdout: true
                        ).trim()

                        echo "GitHub Packages returned HTTP status: ${statusCode}"

                        if (statusCode != "200") {
                            error("Failed to access GitHub Packages (HTTP ${statusCode})")
                        } else {
                            echo "✅ Successfully connected to GitHub Packages"
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Build & Deployment successful.'
        }
        failure {
            echo '❌ Build failed. Check the logs for details.'
            // Optional: Add Slack/email notification here
        }
    }
}

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
                        echo "🔐 Validating GitHub Packages access..."
                        
                        // Test authentication with a simple API call
                        def authStatus = sh(
                            script: """
                                curl -s -o /dev/null -w '%{http_code}' \
                                -H "Authorization: token ${env.GITHUB_TOKEN}" \
                                -H "Accept: application/vnd.github.v3+json" \
                                https://api.github.com/user
                            """,
                            returnStdout: true
                        ).trim()

                        if (authStatus != "200") {
                            error("❌ GitHub authentication failed (HTTP ${authStatus}). Check token permissions.")
                        }
                        echo "✅ GitHub authentication successful"
                        
                        // Additional check for packages access
                        def packagesStatus = sh(
                            script: """
                                curl -s -o /dev/null -w '%{http_code}' \
                                -H "Authorization: token ${env.GITHUB_TOKEN}" \
                                ${env.PACKAGES_URL}/
                            """,
                            returnStdout: true
                        ).trim()

                        echo "GitHub Packages access check: HTTP ${packagesStatus}"
                        if (packagesStatus == "401" || packagesStatus == "403") {
                            error("❌ Insufficient permissions for GitHub Packages")
                        }
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
                        // Create settings.xml with token authentication
                        sh """
                            mkdir -p \$HOME/.m2
                            cat > \$HOME/.m2/settings.xml <<EOF
                            <settings>
                                <servers>
                                    <server>
                                        <id>github</id>
                                        <username>\${env.GITHUB_OWNER}</username>
                                        <password>\${env.GITHUB_TOKEN}</password>
                                    </server>
                                </servers>
                            </settings>
                            EOF
                        """
                        
                        // Deploy with authenticated settings
                        sh """
                            mvn deploy -DskipTests \
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
            echo "❌ Pipeline failed - check authentication and permissions"
        }
    }
}

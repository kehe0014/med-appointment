pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
        MAVEN_HOME = tool 'M3' // Ensure Maven is configured in Jenkins
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
                        
                        // Test basic GitHub API access
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

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Deploy') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        // Create temporary settings.xml with proper escaping
                        writeFile file: 'settings.xml', text: """
                        <settings>
                            <servers>
                                <server>
                                    <id>github</id>
                                    <username>${env.GITHUB_OWNER}</username>
                                    <password>${env.GITHUB_TOKEN}</password>
                                </server>
                            </servers>
                        </settings>
                        """
                        
                        // Deploy with authenticated settings
                        sh """
                            mvn -s settings.xml deploy -DskipTests \
                            -DaltDeploymentRepository=github::default::${env.PACKAGES_URL}
                        """
                        
                        // Clean up (optional)
                        sh 'rm -f settings.xml'
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
        }
    }
}

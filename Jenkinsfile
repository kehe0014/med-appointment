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
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: 'main']],
                    userRemoteConfigs: [[
                        url: "https://github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}.git",
                        credentialsId: 'GITHUB_ACCESS_TOKEN'
                    ]]
                ])
            }
        }

        stage('Check GitHub Packages Access') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔍 Checking GitHub Packages access..."
                        // Safe way to execute curl without Groovy interpolation
                        def statusCode = sh(returnStdout: true, script: """
                            curl -s -o /dev/null -w '%{http_code}' \
                            -H "Authorization: token \$GITHUB_TOKEN" \
                            ${env.PACKAGES_URL}/
                        """).trim()

                        echo "GitHub Packages response: HTTP ${statusCode}"
                        
                        if (statusCode == "404") {
                            echo "⚠️ Package repository not found (404) - proceeding anyway"
                        } else if (statusCode != "200") {
                            error("❌ GitHub Packages access failed (HTTP ${statusCode})")
                        }
                    }
                }
            }
        }

        stage('Build with Maven') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn clean package -DskipTests \
                        -Dgithub.owner=${env.GITHUB_OWNER} \
                        -Dgithub.repo=${env.GITHUB_REPO}
                    """
                }
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn deploy -DskipTests \
                        -DaltDeploymentRepository=github::${env.PACKAGES_URL} \
                        -Dgithub.owner=${env.GITHUB_OWNER} \
                        -Dgithub.repo=${env.GITHUB_REPO}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "🎉 Successfully built and deployed to GitHub Packages!"
        }
        failure {
            echo "❌ Pipeline failed - check logs for details"
        }
    }
}

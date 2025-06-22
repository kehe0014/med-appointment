pipeline {
    agent any

    environment {
        GITHUB_OWNER = 'kehe0014'
        GITHUB_REPO = 'med-appointment'
        PACKAGES_URL = "https://maven.pkg.github.com/${env.GITHUB_OWNER}/${env.GITHUB_REPO}"
        MAVEN_OPTS = "-Dmaven.wagon.http.ssl.insecure=true -Dmaven.wagon.http.ssl.allowall=true"
    }

    stages {
        stage('Check GitHub Packages Access') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    script {
                        echo "🔍 Checking GitHub Packages access..."
                        def statusCode = sh(returnStdout: true, script: """
                            curl -s -o /dev/null -w '%{http_code}' \
                            -H "Authorization: token ${env.GITHUB_TOKEN}" \
                            ${env.PACKAGES_URL}/
                        """).trim()

                        if (statusCode != "200") {
                            error("❌ GitHub Packages inaccessible (HTTP ${statusCode})")
                        }
                        echo "✅ GitHub Packages accessible"
                    }
                }
            }
        }

        stage('Build with Maven') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn clean package \
                        -DskipTests \
                        -Dgithub.owner=${env.GITHUB_OWNER} \
                        -Dgithub.repo=${env.GITHUB_REPO} \
                        -Dgithub.token=${env.GITHUB_TOKEN}
                    """
                }
            }
        }

        stage('Deploy to GitHub Packages') {
            steps {
                withCredentials([string(credentialsId: 'GITHUB_ACCESS_TOKEN', variable: 'GITHUB_TOKEN')]) {
                    sh """
                        mvn deploy \
                        -DskipTests \
                        -DaltDeploymentRepository=github::${env.PACKAGES_URL} \
                        -Dgithub.owner=${env.GITHUB_OWNER} \
                        -Dgithub.repo=${env.GITHUB_REPO} \
                        -Dgithub.token=${env.GITHUB_TOKEN}
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

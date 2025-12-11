pipeline {
    agent any

    stages {

        stage('Detect Branch') {
            steps {
                script {
                    BRANCH = env.GIT_BRANCH ?: sh(
                        script: "git rev-parse --abbrev-ref HEAD",
                        returnStdout: true
                    ).trim()

                    echo "🔎 Current branch: ${BRANCH}"
                }
            }
        }

        /* ========== TEST cho cả 2 nhánh ========== */

        stage('Test') {
            when {
                anyOf {
                    expression { BRANCH == "main" }
                    expression { BRANCH == "Stage-file-JSON" }
                }
            }
            steps {
                echo " Running Tests..."
                sh 'echo "run test here..."'
            }
        }

        /* ========== DOCKER BUILD chỉ cho main ========== */

        stage('Docker Build') {
            when {
                expression { BRANCH == "main" }
            }
            steps {
                script {
                    echo " Building Docker image from src/service/Dockerfile..."

                    sh """
                        docker build \
                            -f src/service/Dockerfile \
                            -t myapp:latest \
                            src/service
                    """
                }
            }
        }
    }
}

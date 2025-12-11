pipeline {
    agent {label 'lab'}

    stages {

        stage('Detect Branch') {
            steps {
                script {
                    BRANCH = env.BRANCH_NAME
                    echo "🔎 Current branch: ${BRANCH}"
                }
            }
        }

        /* ========== TEST cho cả 2 nhánh ========== */
        stage('Test') {
            when {
                expression { env.BRANCH_NAME in ["main", "Stage-file-JSON"] }
            }
            steps {
                echo "🧪 Running Tests..."
                sh 'echo "run test here..."'
            }
        }

        /* ========== DOCKER BUILD chỉ cho main ========== */
        stage('Docker Build') {
            when {
                expression { env.BRANCH_NAME == "main" }
            }
            steps {
                script {

                    echo " Đọc danh sách service trong thư mục src/ ..."

                    // Lấy tất cả folder trong src
                    def services = sh(
                        script: "ls -1 src",
                        returnStdout: true
                    ).trim().split("\n")

                    echo " Danh sách service: ${services}"

                    // Chỉ lấy 2 service đầu tiên nếu có
                    def targets = services.take(2)

                    echo " Sẽ build 2 service đầu tiên: ${targets}"

                    targets.each { svc ->

                        def dockerfilePath = "src/${svc}/Dockerfile"

                        if (!fileExists(dockerfilePath)) {
                            echo " Bỏ qua ${svc} vì không có Dockerfile"
                            return
                        }

                        echo " Building Docker image for: ${svc}"

                        sh """
                            docker build \
                                -f ${dockerfilePath} \
                                -t ${svc}:latest \
                                src/${svc}
                        """
                    }
                }
            }
        }
    }
}

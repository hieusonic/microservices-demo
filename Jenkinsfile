pipeline {
    agent { label 'lab' }

    stages {
        stage('Detect Changes') {
            steps {
                script {

                    // Lấy danh sách file thay đổi
                    def changed = sh(
                        script: "git diff --name-only HEAD~1 HEAD",
                        returnStdout: true
                    ).trim().split("\n")

                    echo "Changed files:\n${changed}"

                    // Tập hợp service cần build
                    servicesToBuild = [] as Set

                    changed.each { file ->
                        // Kiểm tra file thuộc src/<service>/
                        if (file.startsWith("src/")) {
                            def parts = file.split("/")
                            if (parts.size() >= 2) {
                                def service = parts[1]  // ví dụ src/frontend/app.js → frontend
                                servicesToBuild << service
                            }
                        }
                    }

                    if (servicesToBuild.isEmpty()) {
                        echo "⚠️ No changes detected inside /src, skipping build."
                        currentBuild.result = 'SUCCESS'
                        sh "exit 0"
                    }

                    echo "Services to build: ${servicesToBuild}"
                }
            }
        }

        stage('Build Services') {
            when {
                expression { return servicesToBuild && servicesToBuild.size() > 0 }
            }
            steps {
                script {
                    def parallelStages = [:]

                    servicesToBuild.each { svc ->
                        parallelStages["Build ${svc}"] = {
                            stage("Build ${svc}") {
                                def path = "src/${svc}/Dockerfile"

                                if (!fileExists(path)) {
                                    echo "⚠️ Service '${svc}' does not have a Dockerfile → skipping."
                                    return
                                }

                                sh """
                                    echo "🔨 Building ${svc}"
                                    docker build -t ${svc}:latest src/${svc}
                                """
                            }
                        }
                    }

                    parallel parallelStages
                }
            }
        }
    }
}

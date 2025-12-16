pipeline {
    agent { label 'lab' }

    stages {

        stage('Detect Context') {
            steps {
                script {
                    def branchEnvMap = readJSON file: 'branch-env-map.json'
                    def pipelineMap  = readJSON file: 'pipeline-map.json'
                    def serviceMap   = readJSON file: 'services.json'

                    def branch = env.BRANCH_NAME
                    ENV = null

                    // Detect ENV từ branch name
                    branchEnvMap.each { k, v ->
                        if (branch == k || branch.startsWith("${k}/")) {
                            ENV = v
                        }
                    }

                    if (!ENV) {
                        echo "⚠️ No ENV mapping for branch: ${branch}"
                        currentBuild.result = 'SUCCESS'
                        return
                    }

                    ACTIVE_STAGES = pipelineMap[ENV]
                    SERVICES = serviceMap.services

                    echo "🌿 Branch        : ${branch}"
                    echo "🌍 ENV           : ${ENV}"
                    echo "🚀 Pipeline stages: ${ACTIVE_STAGES}"
                    echo "📦 Services      : ${SERVICES}"
                }
            }
        }

        stage('Test') {
            when {
                expression { ACTIVE_STAGES?.contains('test') }
            }
            steps {
                echo "🧪 Running tests"
                sh 'echo "run unit test here"'
            }
        }

        stage('Build Services') {
            when {
                expression { ACTIVE_STAGES?.contains('build') }
            }
            steps {
                script {
                    def parallelBuilds = [:]

                    SERVICES.each { svc ->
                        parallelBuilds["Build ${svc}"] = {
                            stage("Build ${svc}") {
                                def path = "src/${svc}/Dockerfile"

                                if (!fileExists(path)) {
                                    echo "⚠️ ${svc} has no Dockerfile → skip"
                                    return
                                }

                                sh """
                                    echo "🐳 Building ${svc}"
                                    docker build -t ${svc}:${ENV} src/${svc}
                                """
                            }
                        }
                    }

                    parallel parallelBuilds
                }
            }
        }
    }
}

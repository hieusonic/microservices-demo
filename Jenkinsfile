pipeline {
    agent { label 'lab' }

    environment {
        PIPELINE_ENV  = ''
        ACTIVE_STAGES = ''
        SERVICES      = ''
    }

    stages {

        stage('Detect Context') {
            steps {
                script {
                    def branchEnvMap = readJSON file: 'branch-env-map.json'
                    def pipelineMap  = readJSON file: 'pipeline-map.json'
                    def serviceMap   = readJSON file: 'services.json'

                    def branch = env.BRANCH_NAME ?: 'develop'

                    def detectedEnv = null
                    branchEnvMap.each { k, v ->
                        if (branch == k || branch.startsWith("${k}/")) {
                            detectedEnv = v
                        }
                    }

                    if (!detectedEnv) {
                        echo "⚠️ No ENV mapping for branch: ${branch}"
                        currentBuild.result = 'SUCCESS'
                        return
                    }

                    if (!pipelineMap.containsKey(detectedEnv)) {
                        error "❌ ENV '${detectedEnv}' chưa có trong pipeline-map.json"
                    }

                    env.PIPELINE_ENV  = detectedEnv
                    env.ACTIVE_STAGES = pipelineMap[detectedEnv].toList().join(',')
                    env.SERVICES      = serviceMap.services.toList().join(',')

                    echo "🌿 Branch         : ${branch}"
                    echo "🌍 ENV            : ${env.PIPELINE_ENV}"
                    echo "🚀 Pipeline stages: ${env.ACTIVE_STAGES}"
                    echo "📦 Services       : ${env.SERVICES}"
                }
            }
        }

        stage('Test') {
            when {
                expression {
                    env.ACTIVE_STAGES.split(',').contains('test')
                }
            }
            steps {
                echo "🧪 Running tests for ENV=${env.PIPELINE_ENV}"
                sh 'echo "run unit test here"'
            }
        }

        stage('Build Services') {
            when {
                expression {
                    env.ACTIVE_STAGES.split(',').contains('build')
                }
            }
            steps {
                script {
                    def services = env.SERVICES.split(',')
                    def builds = [:]

                    services.each { svc ->
                        builds["Build ${svc}"] = {
                            stage("Build ${svc}") {
                                def dockerfile = "src/${svc}/Dockerfile"

                                if (!fileExists(dockerfile)) {
                                    echo "⚠️ ${svc}: no Dockerfile → skip"
                                    return
                                }

                                sh """
                                    echo "🐳 Building ${svc}"
                                    docker build -t ${svc}:${env.PIPELINE_ENV} src/${svc}
                                """
                            }
                        }
                    }

                    parallel builds
                }
            }
        }
    }
}

pipeline {
    agent { label 'lab' }

    stages {

        stage('Detect Context') {
            steps {
                script {
                    def branchEnvMap = readJSON(
                        file: 'branch-env-map.json',
                        returnPojo: true
                    )

                    def pipelineMap = readJSON(
                        file: 'pipeline-map.json',
                        returnPojo: true
                    )

                    def servicesCfg = readJSON(
                        file: 'services.json',
                        returnPojo: true
                    )

                    def branch = env.BRANCH_NAME

                    if (!branchEnvMap.containsKey(branch)) {
                        error "❌ No ENV mapped for branch: ${branch}"
                    }

                    env.RUN_ENV = branchEnvMap[branch]

                    def stagesToRun = pipelineMap[env.RUN_ENV]
                    if (!(stagesToRun instanceof List)) {
                        error "❌ pipeline-map.json: ENV must map to list"
                    }

                    def services = servicesCfg.services
                    if (!(services instanceof List)) {
                        error "❌ services.json: services must be list"
                    }

                    env.PIPELINE_STAGES = stagesToRun.join(',')
                    env.SERVICES        = services.join(',')

                    echo "🌿 Branch : ${branch}"
                    echo "🌍 ENV    : ${env.RUN_ENV}"
                    echo "🚦 Stages : ${env.PIPELINE_STAGES}"
                    echo "📦 Svcs   : ${env.SERVICES}"
                }
            }
        }

        stage('Test') {
            when {
                expression {
                    env.PIPELINE_STAGES.split(',').contains('test')
                }
            }
            steps {
                echo "🧪 Running tests"
            }
        }

        stage('Build Services') {
            when {
                expression {
                    env.PIPELINE_STAGES.split(',').contains('build')
                }
            }
            steps {
                script {
                    env.SERVICES.split(',').each { svc ->
                        stage("Build ${svc}") {
                            def dockerfile = "src/${svc}/Dockerfile"

                            if (!fileExists(dockerfile)) {
                                echo "⚠️ ${svc}: no Dockerfile → skip"
                                return
                            }

                            sh """
                                echo "🐳 Building ${svc}"
                                docker build -t ${svc}:${env.RUN_ENV} src/${svc}
                            """
                        }
                    }
                }
            }
        }
    }
}

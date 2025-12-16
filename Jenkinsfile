pipeline {
    agent { label 'lab' }

    stages {

        stage('Detect Context') {
            steps {
                script {
                    // Read config files
                    def branchEnvMap = readJSON file: 'branch-env-map.json'
                    def pipelineMap  = readJSON file: 'pipeline-map.json'
                    def servicesCfg  = readJSON file: 'services.json'

                    def branch = env.BRANCH_NAME

                    // Map branch -> ENV
                    if (!branchEnvMap.containsKey(branch)) {
                        error "❌ No ENV mapped for branch: ${branch}"
                    }

                    env.RUN_ENV = branchEnvMap[branch]

                    // IMPORTANT FIX: pipeline-map has key "stages"
                    def rawStages = pipelineMap[env.RUN_ENV].stages
                    def activeStages = rawStages.collect { it.toString() }

                    def services = servicesCfg.services.collect { it.toString() }

                    // Store as env string (safe for declarative when)
                    env.PIPELINE_STAGES = activeStages.join(',')
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
                sh 'echo "run unit test here"'
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

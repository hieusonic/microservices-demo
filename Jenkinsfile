pipeline {
    agent { label 'lab' }

    options {
        skipDefaultCheckout(true)
    }

    stages {

        /* =========================
           1. Checkout
        ========================== */
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        /* =========================
           2. Detect Context
        ========================== */
        stage('Detect Context') {
            steps {
                script {
                    // Read JSON files (return Map/List thuần)
                    def branchEnvMap = readJSON(
                        file: 'branch-env-map.json',
                        returnPojo: true
                    )

                    def pipelineMap = readJSON(
                        file: 'pipeline-map.json',
                        returnPojo: true
                    )

                    def servicesConfig = readJSON(
                        file: 'services.json',
                        returnPojo: true
                    )

                    // Detect branch
                    def branch = env.BRANCH_NAME
                    if (!branchEnvMap.containsKey(branch)) {
                        error "❌ No environment mapped for branch: ${branch}"
                    }

                    // Resolve ENV
                    env.RUN_ENV = branchEnvMap[branch]

                    // Resolve stages to run
                    def stagesToRun = pipelineMap[env.RUN_ENV]
                    if (!stagesToRun) {
                        error "❌ No pipeline stages defined for env: ${env.RUN_ENV}"
                    }

                    // Resolve services
                    def services = servicesConfig.services ?: []

                    // Export to env (string only)
                    env.PIPELINE_STAGES = stagesToRun.join(',')
                    env.SERVICES = services.join(',')

                    echo "✅ Branch      : ${branch}"
                    echo "✅ Environment : ${env.RUN_ENV}"
                    echo "✅ Stages      : ${env.PIPELINE_STAGES}"
                    echo "✅ Services    : ${env.SERVICES}"
                }
            }
        }

        /* =========================
           3. Test
        ========================== */
        stage('Test') {
            when {
                expression {
                    env.PIPELINE_STAGES.split(',').contains('test')
                }
            }
            steps {
                echo "🧪 Running tests for ENV=${env.RUN_ENV}"
                // sh 'make test' (ví dụ)
            }
        }

        /* =========================
           4. Build Services
        ========================== */
        stage('Build Services') {
            when {
                expression {
                    env.PIPELINE_STAGES.split(',').contains('build')
                }
            }
            steps {
                script {
                    def services = env.SERVICES.split(',')

                    services.each { svc ->
                        stage("Build ${svc}") {
                            def dockerfile = "src/${svc}/Dockerfile"

                            if (!fileExists(dockerfile)) {
                                error "❌ Dockerfile not found for service: ${svc}"
                            }

                            echo "🐳 Building ${svc}"
                            sh """
                                docker build -t ${svc}:latest src/${svc}
                            """
                        }
                    }
                }
            }
        }
    }
}

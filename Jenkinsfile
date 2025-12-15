def SERVICES = []
def PIPELINE_STAGES = []

pipeline {
    agent { label 'lab' }

    parameters {
        choice(
            name: 'ENV',
            choices: ['dev', 'uat'],
            description: 'Select environment'
        )
    }

    options {
        timestamps()
    }

    stages {

        stage('Load Config') {
            steps {
                script {
                    def serviceCfg  = readJSON file: 'services.json'
                    def pipelineCfg = readJSON file: 'pipeline-map.json'

                    SERVICES = serviceCfg.services

                    if (!pipelineCfg.containsKey(params.ENV)) {
                        error "❌ ENV '${params.ENV}' not found in pipeline-map.json"
                    }

                    PIPELINE_STAGES = pipelineCfg[params.ENV].stages

                    echo "🌍 Environment: ${params.ENV}"
                    echo "📦 Services: ${SERVICES}"
                    echo "🧩 Stages for ENV: ${PIPELINE_STAGES}"
                }
            }
        }

        stage('Build & Test Services') {
            when {
                expression { SERVICES && SERVICES.size() > 0 }
            }
            steps {
                script {
                    def parallelStages = [:]

                    SERVICES.each { svc ->
                        parallelStages[svc] = {
                            stage("Service: ${svc}") {

                                def dockerfilePath = "src/${svc}/Dockerfile"

                                if (!fileExists(dockerfilePath)) {
                                    echo "⚠️ ${svc} has no Dockerfile → skipping"
                                    return
                                }

                                if (PIPELINE_STAGES.contains("build")) {
                                    sh """
                                        echo "🔨 Building ${svc}"
                                        docker build -t ${svc}:latest src/${svc}
                                    """
                                }

                                if (PIPELINE_STAGES.contains("test")) {
                                    sh """
                                        echo "🧪 Testing ${svc}"
                                        echo "(demo test for ${svc})"
                                    """
                                }
                            }
                        }
                    }

                    parallel parallelStages
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline SUCCESS for ENV=${params.ENV}"
        }
        failure {
            echo "❌ Pipeline FAILED for ENV=${params.ENV}"
        }
    }
}

pipeline {
    agent { label 'lab' }

    options {
        timestamps()
    }

    stages {

        stage('Load Config') {
            steps {
                script {
                    def serviceCfg = readJSON file: 'services.json'
                    def pipelineCfg = readJSON file: 'pipeline-map.json'

                    SERVICES = serviceCfg.services
                    PIPELINE_STAGES = pipelineCfg.stages

                    echo "📦 Services to build: ${SERVICES}"
                    echo "🧩 Pipeline stages: ${PIPELINE_STAGES}"
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
                                        echo "(demo test step for ${svc})"
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
            echo "✅ Pipeline completed successfully"
        }
        failure {
            echo "❌ Pipeline failed"
        }
    }
}

pipeline {
    agent any
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        DETECT_PROJECT_NAME = "${env.REPO_NAME}"
        DETECT_PROJECT_VERSION_NAME = "$BRANCH_NAME"
        DETECT_CODE_LOCATION_NAME = "${env.REPO_NAME}-$BRANCH_NAME"
        DETECT_RISK_REPORT_PDF = true
    }
    stages {
        stage('build') {
            steps {
                sh 'mvn -B package'
            }
        }
        stage('blackduck') {
            steps {
                script {
                    status = synopsys_scan product: 'blackduck',
                        blackduck_scan_full: true,
                        blackduck_scan_failure_severities: 'BLOCKER',
                        blackduck_reports_sarif_create: true
                    if (status == 8) { unstable 'policy violation' }
                    else if (status != 0) { error 'plugin failure' }
                }
            }
        }
    }
    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: '.bridge/bridge.log, *_BlackDuck_RiskReport.pdf'
            //zip archive: true, dir: '.bridge', zipFile: 'bridge-logs.zip'
            cleanWs()
        }
    }
}

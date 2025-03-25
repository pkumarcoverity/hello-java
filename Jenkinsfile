pipeline {
    agent any
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        FULLSCAN = "${env.BRANCH_NAME ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
        PRSCAN = "${env.CHANGE_TARGET ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
        GITHUB_TOKEN = credentials('github-pkumarcoverity')
        DETECT_PROJECT_NAME = "${env.REPO_NAME}"
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn -B package'
            }
        }
        stage('Black Duck') {
            when {
                anyOf {
                    environment name: 'FULLSCAN', value: 'true'
                    environment name: 'PRSCAN', value: 'true'
                }
            }
            steps {
                security_scan product: 'blackducksca',
                    blackducksca_scan_failure_severities: 'BLOCKER',
                    blackducksca_prComment_enabled: false,
                    blackducksca_reports_sarif_create: false,
                    mark_build_status: 'UNSTABLE',
                    github_token: "$GITHUB_TOKEN",
                    include_diagnostics: false
            }
        }
    }
    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: '.bridge/bridge.log'
            //zip archive: true, dir: '.bridge', zipFile: 'bridge-logs.zip'
            cleanWs()
        }
    }
}

pipeline {
    agent { label 'linux64' }
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        FULLSCAN = "${env.BRANCH_NAME ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
        PRSCAN = "${env.CHANGE_TARGET ==~ /^(main|master|develop|stage|release)$/ ? 'true' : 'false'}"
        GITHUB_TOKEN = credentials('github-pat')
        DETECT_PROJECT_NAME = "${env.REPO_NAME}"
    }
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
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
                    blackduckscasca_scan_failure_severities: 'BLOCKER',
                    blackduckscasca_prComment_enabled: true,
                    blackduckscasca_reports_sarif_create: true,
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

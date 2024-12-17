pipeline {
    agent { label 'linux64' }
    tools {
        maven 'maven-3.9'
        jdk 'openjdk-17'
    }
    environment {
        REPO_NAME = "${env.GIT_URL.tokenize('/.')[-2]}"
        COV_URL = 'http://20.39.40.145:8090'
        COVERITY_CREDS = credentials('azure-coverity')
        COV_USER = "admin"
        COVERITY_PASSPHRASE = "SIGpass8!"
        COV_PROJECT = "pkumarb-sec-coverity"
        COV_STREAM = "pkumarb-sec-coverity"
    }
    stages {
        stage('checkout') {
            steps {
                git url: "$GIT_URL", branch: "$BRANCH_NAME", credentialsId: 'github-pkumarcoverity'
            }
        }
        stage('coverity') {
            steps {
                sh '''
                    $COVERITY_TOOL_HOME/bin/coverity --ticker-mode none scan \
                        -o commit.connect.url=$COV_URL -o commit.connect.project=$COV_PROJECT -o commit.connect.stream=$COV_STREAM \
                        -o commit.connect.description=$BUILD_TAG -o commit.connect.version=\$(git log -n 1 --pretty=format:%H)
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts allowEmptyArchive: true, artifacts: 'idir/build-log.txt, idir/output/analysis-log.txt'
            cleanWs()
        }
    }
}

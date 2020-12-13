def project_name = "${env.JOB_NAME}".replace("%2F", "/")

pipeline {
  agent any

  options {
    buildDiscarder(logRotator(numToKeepStr: '100'))
  }

  stages {
    stage('Install Prerequisites') {
      parallel {
        stage('Notify slack') {
          steps {
            slackSend channel: 'chatops', message: "${project_name} - #${env.BUILD_NUMBER} Started build (<${env.RUN_DISPLAY_URL}|Open>)"
          }
        }
        stage('yarn deps') {
          steps {
            sh 'yarn'
          }
        }
      }
    }

    stage('Build and Package') {
      steps {
        sh 'yarn build'
        sh '''BRANCH="$(echo ${GIT_BRANCH} | tr \'/\' \'_\' | cut -d \'_\' -f 2-)"
              TREEISH="$(echo ${GIT_COMMIT} | cut -c 1-6)"
              FN="bachelor-webapp-b${BUILD_NUMBER}-${BRANCH}@${TREEISH}.tar.xz"
              export XZ_OPT="-3 -T2"

              tar --exclude-vcs --exclude-vcs-ignores -C "${WORKSPACE}/build" -cJf "/tmp/${FN}" "."
              mv "/tmp/${FN}" .'''
      }
    }

    stage('Archive and Upload') {
      steps {
         s3Upload consoleLogLevel: 'INFO', dontWaitForConcurrentBuildCompletion: false, dontSetBuildResultOnFailure: false, entries: [[bucket: 'builds', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'xvolve-internal-1', showDirectlyInBrowser: false, sourceFile: '*xz', storageClass: 'STANDARD', uploadFromSlave: true, useServerSideEncryption: false, userMetadata: [[key: 'branch', value: '${GIT_BRANCH}'], [key: 'treeish', value: '${GIT_COMMIT}'], [key: 'build-number', value: '${BUILD_NUMBER}'], [key: 'project', value: '${JOB_NAME}']]]], pluginFailureResultConstraint: 'FAILURE', profileName: 'xvolve-internal-1', userMetadata: []
      }
    }

    stage('Test') {
      steps {
        sh 'yarn run test || true'
      }
    }

  }

  post {
    success {
      slackSend channel: 'chatops', color: 'good', message: "${project_name} - #${env.BUILD_NUMBER} Build Success (<${env.RUN_DISPLAY_URL}|Open>). Took ${currentBuild.durationString}.\nChanges: \n$changeString"
    }
    unstable {
      slackSend channel: 'chatops', color: 'warning', message: "${project_name} - #${env.BUILD_NUMBER} Build Completed, UNSTABLE (<${env.RUN_DISPLAY_URL}|Open>). Took ${currentBuild.durationString}.\nChanges: \n$changeString"
    }
    failure {
      slackSend channel: 'chatops', color: 'danger', message: "${project_name} - #${env.BUILD_NUMBER} Build Failure (<${env.RUN_DISPLAY_URL}|Open>). Took ${currentBuild.durationString}.\nChanges: \n$changeString"
    }
    always {
      junit 'junit.xml'
      cucumber buildStatus: 'UNSTABLE',
        fileIncludePattern: 'report.json',
        sortingMethod: 'ALPHABETICAL',
        trendsLimit: 100
      publishHTML (target: [
        allowMissing: false,
        alwaysLinkToLastBuild: false,
        keepAll: true,
        reportDir: 'coverage/lcov-report',
        reportFiles: 'index.html',
        reportName: "lCov Report"
      ])
      cleanWs()
    }
  }

}

@NonCPS
def getChangeString() {
    MAX_MSG_LEN = 100
    def changeString = ""
    echo "Gathering SCM changes"
    def changeLogSets = currentBuild.rawBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            truncated_msg = entry.msg.take(MAX_MSG_LEN)
            changeString += " - ${truncated_msg} [${entry.author}]\n"
        }
    }
    if (!changeString) {
        changeString = " - No new changes"
    }
    return changeString
}

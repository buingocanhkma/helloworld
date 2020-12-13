/** PROJECT PROPERTIES
* project: Project Name
* gitUrl: Link repository
* branch: Branch Name exist in repository
* credentialID: Login to repository
*/
def project = "helloworld"
def gitUrl = "https://github.com/buingocanhkma/helloworld.git"
def branch = "develop"
def credentialID = "anhbn_github"


pipeline {

    /** agent
    * label: Agent name will execute pipeline
    */
    agent {
        label 'master'
    }
    /** Checkout
    * Get source code from SVN, Git,...
    */
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "${branch}"]],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CleanBeforeCheckout']], gitTool: 'jgitapache',
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: "${credentialID}",
                        url: "${gitUrl}"]]
                ])
            }
        }

        /** Build - Compile
        * writeFile: Write file build commandline
        * file: File name
        * text: Commandline build
        * build.bat: Build commandline use ant build tool
        */
        stage('Build') {
            steps {
                sh "ls -al"
            }
        }
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
              FN="helloworld-b${BUILD_NUMBER}-${BRANCH}@${TREEISH}.tar.xz"
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
		
    }
}


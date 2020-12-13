/** PROJECT PROPERTIES
* project: Project Name
* gitUrl: Link repository
* branch: Branch Name exist in repository
* credentialID: Login to repository
*/
def project = "helloworld"
def gitUrl = "https://github.com/buingocanhkma/helloworld.git"
def branch = "master"
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
		
    }
}


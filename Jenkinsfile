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
		
    stage('Build Tarball') {
      steps {
        sh '''BRANCH="$(echo ${GIT_BRANCH} | tr \'/\' \'_\' | cut -d \'_\' -f 2-)"
TREEISH="$(echo ${GIT_COMMIT} | cut -c 1-6)"
FN="helloworld-v1-b${BUILD_NUMBER}-${BRANCH}@${TREEISH}.tar.xz"
echo "building ${FN}"
#build deployable archive:
export XZ_OPT="-3 -T2"
tar --exclude-vcs --exclude-vcs-ignores -cJf "/tmp/${FN}" -C "${WORKSPACE}" .
mv "/tmp/${FN}" .'''
      }
    }
	
    stage('Install Prerequisites') {
      parallel {
        stage('yarn deps') {
          steps {
            sh 'yarn'
          }
        }
      }
    }

    stage('Archive and Upload') {
      steps {
        s3Upload consoleLogLevel: 'INFO', dontWaitForConcurrentBuildCompletion: false, dontSetBuildResultOnFailure: false, entries: [[bucket: 'builds', excludedFile: '', flatten: false, gzipFiles: false, keepForever: false, managedArtifacts: true, noUploadOnFailure: true, selectedRegion: 'xvolve-internal-1', showDirectlyInBrowser: false, sourceFile: '*xz', storageClass: 'STANDARD', uploadFromSlave: true, useServerSideEncryption: false, userMetadata: [[key: 'branch', value: '${GIT_BRANCH}'], [key: 'treeish', value: '${GIT_COMMIT}'], [key: 'build-number', value: '${BUILD_NUMBER}'], [key: 'project', value: '${JOB_NAME}']]]], pluginFailureResultConstraint: 'FAILURE', profileName: 'xvolve-internal-1', userMetadata: []
      }
    }
		
    }
	
    stage('Prepare For Testing') {
      environment {
        TESTDB_CRED = credentials('dlsgmaria-maint-user')
      }
      steps {
        sh './scripts/ci/create-temp-dbs.sh 14'
        sh 'mkdir junit'
      }
    }

    stage('Test') {
      parallel {
        stage('[behat] Participation') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-1'
              sh 'APP_ENV=test-1 ./vendor/bin/behat -f pretty -o storage/logs/behat-participation.log -f junit -o junit/1 -f cucumber_json -o junit/1 -s Participation --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Dating Metadata') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-2'
              sh 'APP_ENV=test-2 ./vendor/bin/behat -f pretty -o storage/logs/behat-datingmeta.log -f junit -o junit/2 -f cucumber_json -o junit/2 -s DatingMetadata --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Dating Cancellation') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-3'
              sh 'APP_ENV=test-3 ./vendor/bin/behat -f pretty -o storage/logs/behat-datingcancel.log -f junit -o junit/3 -f cucumber_json -o junit/3 -s DatingCancellation --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] TuesdayMatching') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'APP_ENV=test-4 ./vendor/bin/behat -f pretty -o storage/logs/behat-tuesdaymatching.log -f junit -o junit/4 -f cucumber_json -o junit/4 -s TuesdayMatching --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] ThursdayMatching') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'APP_ENV=test-5 ./vendor/bin/behat -f pretty -o storage/logs/behat-thursdaymatching.log -f junit -o junit/5 -f cucumber_json -o junit/5 -s ThursdayMatching --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Subscriptions') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-6'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-6'
              sh 'APP_ENV=test-6 ./vendor/bin/behat -f pretty -o storage/logs/behat-subscriptions.log -f junit -o junit/6 -f cucumber_json -o junit/6 -s Subscriptions --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Notifications') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-7'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-7'
              sh 'php artisan db:seed --class=AnnualIncomeTableSeeder --env=test-7'
              sh 'APP_ENV=test-7 ./vendor/bin/behat -f pretty -o storage/logs/behat-notifications.log -f junit -o junit/7 -f cucumber_json -o junit/7 -s Notifications --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Feedback') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-8'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-8'
              sh 'APP_ENV=test-8 ./vendor/bin/behat -f pretty -o storage/logs/behat-feedback.log -f junit -o junit/8 -f cucumber_json -o junit/8 -s Feedback --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] AutomateRematching') {
         steps {
           warnError('Suite ran unsuccessfully 🤕') {
             sh 'APP_ENV=test-9 ./vendor/bin/behat -f pretty -o storage/logs/behat-automate-rematching.log -f junit -o junit/9 -f cucumber_json -o junit/9 -s AutomateRematching --no-snippets --verbose=2 --no-colors'
           }
         }
        }
        stage('[behat] Invitation') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-10'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-10'
              sh 'APP_ENV=test-10 ./vendor/bin/behat -f pretty -o storage/logs/behat-invitation.log -f junit -o junit/10 -f cucumber_json -o junit/10 -s Invitation --no-snippets --verbose=2 --no-colors'
           }
          }
        }
        stage('[behat] DeactivateAccount') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-11'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-11'
              sh 'APP_ENV=test-11 ./vendor/bin/behat -f pretty -o storage/logs/behat-deactivation-account.log -f junit -o junit/11 -f cucumber_json -o junit/11 -s DeactivateAccount --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] Registration') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-12'
              sh 'APP_ENV=test-12 ./vendor/bin/behat -f pretty -o storage/logs/behat-registration.log -f junit -o junit/12 -f cucumber_json -o junit/12 -s Registration --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] UserCancelDeactive') {
          steps {
            warnError('Suite ran unsuccessfully 🤕') {
              sh 'php artisan migrate --seed --env=test-13'
              sh 'php artisan db:seed --class=SettingNotificationSeeder --env=test-13'
              sh 'APP_ENV=test-13 ./vendor/bin/behat -f pretty -o storage/logs/behat-user-cancel-deactive.log -f junit -o junit/13 -f cucumber_json -o junit/13 -s UserCancelDeactive --no-snippets --verbose=2 --no-colors'
            }
          }
        }
        stage('[behat] SecondAutomateRematching') {
         steps {
           warnError('Suite ran unsuccessfully 🤕') {
             sh 'APP_ENV=test-14 ./vendor/bin/behat -f pretty -o storage/logs/behat-automate-rematching-2.log -f junit -o junit/14 -f cucumber_json -o junit/14 -s AutomateRematching --no-snippets --verbose=2 --no-colors'
           }
         }
        }
      }
    }

    stage('Testing Cleanup') {
      environment {
        TESTDB_CRED = credentials('dlsgmaria-maint-user')
      }
      steps {
        sh './scripts/ci/drop-temp-dbs.sh 14'
        archiveArtifacts(artifacts: 'storage/logs/*', fingerprint: true)
        livingDocs(featuresDir:'junit')
      }
    }

}


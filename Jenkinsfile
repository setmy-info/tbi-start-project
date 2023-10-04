import jenkins.model.Jenkins
import hudson.plugins.git.GitSCM
import hudson.plugins.git.util.DefaultBuildChooser
import hudson.plugins.git.util.BuildChooserContext
import hudson.plugins.git.util.BuildData

pipeline {

    // version 1.0.0

    agent any

    /*
    triggers {
        cron('H/5 * * * *')
    }
    */

    options {
        // Disable at branch level parallel build
        disableConcurrentBuilds()
    }

    environment {
        PATH = "/opt/has/bin:$PATH"
        ABC = 'DEF'
        GHI = "$ABC"

        MASTER_TO_LIVE = 'DEPLOY'

        MASTER_TO_PRELIVE = 'DEPLOY'
        RELEASE_TO_PRELIVE = 'DEPLOY'

        DEVELOPMENT_TO_TESTING = 'DEPLOY'
        RELEASE_TO_TESTING = 'DEPLOY'

        DEVELOPMENT_TO_DEV = 'DEPLOY'
        RELEASE_TO_DEV = 'DEPLOY'
    }

    stages {
        stage('Inspection') {
            parallel {
                stage('Examples') {
                    steps {
                        echo 'That section can be deleted for real life situations'
                        sh 'echo "GHI=${GHI}"'
                        echo "Message GHI=${GHI}"
                        sleep 5
                        retry(count: 7) {
                            sh 'echo "Many times, why?"'
                        }
                        timeout(time: 10) {
                            sh 'echo "What is this time?"'
                            // sh 'exit 1' // Failing that step
                        }
                        script {
                	    def jobName = env.JOB_NAME
                	    def buildNumber = env.BUILD_NUMBER
                	    def previousBuild = currentBuild.previousBuild
			    //def gitChangeSet = currentBuild.changeSets[0]
			    def allChanges = []
     			    for (changeSet in currentBuild.changeSets) {
	                        allChanges.addAll(changeSet.items)
	                    }
			    
			    //def job = Jenkins.instance.getItemByFullName(env.JOB_NAME)
			    //def build = job.getBuildByNumber(buildNumber)
			    print "Job name: $jobName"
			    print "Job number: $buildNumber"
			    print "previousBuild: $previousBuild , allChanges: $allChanges"
			    if (previousBuild != null) {
			        def affectedFilePaths = allChanges.collect { it.paths }
			        print "affectedFilePaths: $affectedFilePaths"
			        def affectedFiles = affectedFilePaths.collect { it.path }
			        print "affectedFiles: $affectedFiles"
			        def tbiFiles = affectedFiles.findAll { 
				        print "File: ${it}"
					def isTbiFile = false
	/*					if (it.startsWith('sprint-backlog/tbi-') && (it.endsWith('.yaml') || it.endsWith('.yml'))) {
						isTbiFile = true
					}*/
					isTbiFile
				}
			        print "Changed file names compared by previous build: ${affectedFiles.join(', ')}"
			        print "TBI files: ${tbiFiles.join(', ')}"
			    } else {
			        print "No changes by previous build or no changes."
			    }
			}
			script {
			   echo "Scripti proov."
			}
                        // build(job: 'has-web-app-new', propagate: true)
                        emailext (
                            subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER",
                            body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}",
                            recipientProviders: [[$class: 'DevelopersRecipientProvider']]
                        )
                        fileExists 'README.md'
                        fileExists 'sprint-backlog/backlog_txt'
                    }
                }
                stage('Build tools') {
                    steps {
                        echo 'Put here commands to check, that build tools are installed'
                        sh 'echo "Hello stage B"'
                    }
                }
                stage('Jenkins Scripting') {
                    steps {
                	script {
                	    print "Hello Scripting."
			}
		   }
                }
            }
        }

        stage('Preparation') {
            parallel {
                stage('Install') {
                    steps {
                        echo 'Installation commands go here'
                        echo 'npm install'
                        echo 'Put here build configuration commands'
                        echo './config'
                    }
                }
            }
        }

        stage('Build') {
            steps {
                echo 'Put here resource copy commands'
                echo 'mvn resources'
                echo 'Put here compilation commands'
                echo 'mvn compile'
                echo 'Put here unit tests'
                echo 'mvn test'
                echo 'Put here unit tests coverage'
                echo 'mvn test'
                echo 'Put here mutation tests coverage'
                echo 'mvn test'
                echo 'Put here integration tests'
                echo 'mvn verify'
                echo 'Put here findbug/stopbug, style check'
                echo 'dependencies vulnreability checks'
                echo 'Put here system tests'
                echo 'Put here acceptance tests'
                echo 'Put here reporting builds steps can include (unit tests coverage, mutation test coverage, findbugs, vuln. checks, )'
                echo 'mvn site'
                echo 'Put here packaging'
                echo 'mvn package'
                echo 'Put here local publishing'
                echo 'mvn install'
            }
        }

        stage('Publish') {
            parallel {
                stage('Release') {
                    when {
                        branch 'master'
                        // changeset "**/file/to/be/changed"
                    }
                    steps {
                        echo 'Put here software release steps'
                    }
                }
                stage('Snapshot') {
                    when {
                        expression { env.BRANCH_NAME.startsWith('devel') }
                    }
                    steps {
                        echo 'Put here software snapshot publishing steps'
                    }
                }
                stage('Release reports') {
                    when {
                        branch 'master'
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
                stage('Snapshot reports') {
                    when {
                        expression { env.BRANCH_NAME.startsWith('devel') }
                    }
                    steps {
                        echo 'Put here reports publishing steps'
                    }
                }
            }
        }
        stage('Deploy') {
            parallel {
                stage('dev') {
                    when {
                        expression {
                            (env.DEVELOPMENT_TO_DEV == 'DEPLOY' && env.BRANCH_NAME.startsWith('devel')) ||
                            (env.RELEASE_TO_DEV == 'DEPLOY' && env.BRANCH_NAME.startsWith('release'))
                        }
                    }
                    steps {
                        echo 'Put here software development installations steps'
                    }
                }
                stage('testing') {
                    when {
                        expression {
                            (env.DEVELOPMENT_TO_TESTING == 'DEPLOY' && env.BRANCH_NAME.startsWith('devel')) ||
                            (env.RELEASE_TO_TESTING == 'DEPLOY' && env.BRANCH_NAME.startsWith('release'))
                        }
                    }
                    steps {
                        echo 'Put here software development installations steps'
                    }
                }
                stage('prelive') {
                    when {
                        expression {
                            env.RELEASE_TO_PRELIVE == 'DEPLOY' && env.BRANCH_NAME.startsWith('release')
                        }
                    }
                    steps {
                        echo 'Put here software prelive installations steps'
                    }
                }
                stage('live') {
                    when {
                        expression {
                            env.MASTER_TO_LIVE == 'DEPLOY' && env.BRANCH_NAME == 'master'
                        }
                    }
                    steps {
                        echo 'Put here software production installations steps'
                    }
                }
            }
        }
        stage('Tag') {
            when {
                branch 'master'
                expression { env.MASTER_TO_LIVE == 'DEPLOY' }
            }
            steps {
                echo 'Put here taging'
            }
        }
    }

    post {
        always {
            // junit '**/target/*-reports/*.xml'
            sh 'echo "Allways"'
        }

        success {
            emailext (
                subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER type: SUCCESSFUL",
                body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}, git: ${env.GIT_URL}, branch: ${env.GIT_BRANCH} SUCCESSFUL post step",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }

        failure {
            emailext (
                subject: "Jenkins job: $JOB_NAME, build: $BUILD_NUMBER type: FAILED",
                body: "Job: $JOB_NAME, build: $BUILD_NUMBER, url: ${env.BUILD_URL}, git: ${env.GIT_URL}, branch: ${env.GIT_BRANCH}  FAILED post step",
                recipientProviders: [[$class: 'DevelopersRecipientProvider']]
            )
        }
    }
}

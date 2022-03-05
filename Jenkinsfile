pipeline {
    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '10'))
        timestamps()
    }

    triggers {
        pollSCM('H/24 * * * *') // once a day in case some hooks are missed
    }

    stages {
        stage('Build') {
            parallel {
                stage('Windows') {
                    agent {
                        label 'docker-windows'
                    }
                    options {
                        timeout(time: 60, unit: 'MINUTES')
                    }
                    environment {
                        DOCKERHUB_ORGANISATION = "${infra.isTrusted() ? 'jenkins' : 'jenkins4eval'}"
                    }
                    steps {
                        powershell '& ./make.ps1 test'
                        script {
                            def branchName = "${env.BRANCH_NAME}"
                            if (branchName ==~ 'master') {
                                // we can't use dockerhub builds for windows
                                // so we publish here
                                infra.withDockerCredentials {
                                    powershell '& ./make.ps1 publish'
                                }
                            }

                            def tagName = "${env.TAG_NAME}"
                            if(tagName =~ /\d(\.\d)+(-\d+)?/) {
                                // we need to build and publish the tagged version
                                infra.withDockerCredentials {
                                    powershell "& ./make.ps1 -PushVersions -VersionTag $tagName publish"
                                }
                            }
                        }
                    }
                    post {
                        always {
                            junit(allowEmptyResults: true, keepLongStdio: true, testResults: 'target/**/junit-results.xml')
                        }
                    }
                }
            }
        }
    }

}

// vim: ft=groovy

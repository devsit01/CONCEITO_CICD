pipeline {
    agent { label "Master" }
    environment {
        // General
        COMPILER = 'C:\\Program Files\\DataFlex 24.0\\Bin64\\DfCompConsole.exe'
    }
    stages {
        stage("Pipeline") {
            parallel {
                stage("Unit Tests") {
                    environment {
                        WORKSPACE_NAME = "DFUnit-Sample"
                        UNITTEST_NAME = "RunAllTests"

                        WORKSPACE_DIR = "DFUnit Example Workspace"
                        WORKSPACE_OUTPUT = "${WORKSPACE_DIR}\\Programs"
                        UNITTEST_SWS = "${WORKSPACE_DIR}\\${WORKSPACE_NAME}.sws"
                        UNITTEST_SRC = "AppSrc\\${UNITTEST_NAME}.src"
                        UNITTEST_JUNIT_OUTPUT = "test_results.xml"
                    }
                    stages {
                        stage("Build") {
                            steps {
                                script {
                                    bat "\"${COMPILER}\" -x\"${UNITTEST_SWS}\" -c \"${UNITTEST_SRC}\""
                                    stash name: 'build', includes: "**"
                                }
                            }
                        }
                        stage('Test') {
                            steps {
                                unstash 'build'
                                powershell(script: """
                                    \$executable = "${WORKSPACE_OUTPUT}\\${UNITTEST_NAME}.exe"
                                    \$arguments = "--console -o ${UNITTEST_JUNIT_OUTPUT}"
                                    \$process = Start-Process -Wait -PassThru \$executable \$arguments
                                    if (\$process.ExitCode -ne 0) {
                                        echo "Unit Tests failed with exit code: \$(\$process.ExitCode)."
                                        exit -1
                                    }
                                """)
                            }
                            post {
                                always {
                                    junit "${UNITTEST_JUNIT_OUTPUT}"
                                    archiveArtifacts artifacts: "${UNITTEST_JUNIT_OUTPUT}", fingerprint: true
                                }
                            }
                        }
                        stage('Archive') {
                            steps {
                                script {
                                    unstash 'build'
                                    bat "tar acvf Build.zip \"${WORKSPACE_OUTPUT}\\*.exe\" \"${WORKSPACE_OUTPUT}\\*.ws\""
                                }
                            }
                            post {
                                always {
                                    archiveArtifacts artifacts: "Build.zip", fingerprint: true
                                }
                            }
                        }

                        stage('Deploy on VADER') {
                            agent { node { label 'TESTE_VADER' } }
                            steps {
                                unstash 'build'
                                bat """
                                    if not exist E:\\Teste mkdir E:\\Teste
                                    copy /Y "${WORKSPACE_OUTPUT}\\*.exe" E:\\Teste\\
                                    copy /Y "${WORKSPACE_OUTPUT}\\*.ws" E:\\Teste\\
                                """
                            }
                        }
                    }
                }
            }
        }
    }
}
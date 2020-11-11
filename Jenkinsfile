pipeline {
    agent {
        docker {
            image 'katalonstudio/katalon'
            args '-u root'
        }
    }

    environment {
        QA_ENV = "${params.Environment}"
        TAG = "${params.Tag}"
		TYPE = "${params.Type}"

        PROFILE = 'DevOps'
        BROWSER = 'Chrome'
        LOGS = 'DevOps logs'
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    if( "${QA_ENV}" == "DIT" ) {
                        ENDPOINT = 'https://coo-acd-dsradevopsdemo-mdx-ui-dit02.azcpggpc.ca/'
                    } else if( "${QA_ENV}" == "QA" ) {
                        ENDPOINT = 'https://coo-acd-dsradevopsdemo-mdx-ui-qa01.azcpggpc.ca/'
                    } else if( "${QA_ENV}" == "BIT50" ) {
                        ENDPOINT = 'https://coo-acd-dsradevopsdemo-mdx-ui-bit50.azcpggpc.ca/'
                    } 
					

                    if( "${TAG}" == "Sanity" ) {
                        PACKAGE = "${TYPE}/DevOps/DevOpsSanity"
                    } else if( "${TAG}" == "Regression" ) {
                        PACKAGE = "${TYPE}/DevOps/DevOpsRegression"
                    } else if( "${TAG}" == "Acceptance" ) {
                        PACKAGE = "${TYPE}/DevOps/DevOpsAcceptance"
                    }

                    TESTSUITE = "Test Suites/${PACKAGE}"
                    OUTPUT = "**/${PACKAGE}/*/*"
                }
            }
        }
        stage('Test') {
            steps {

                sh "katalonc.sh -projectPath=\"${WORKSPACE}\" -g_RP_LAUNCH=\"${JOB_NAME}\" -g_RP_DESC=\"${TAG}\" -g_APP_URL=\"${ENDPOINT}\" -runMode=console -executionProfile=\"${PROFILE}\" -browserType=\"${BROWSER}\" -retry=0 -testSuitePath=\"${TESTSUITE}\" -apikey=ef3b2cd2-a2ec-4e1f-baa2-ccfbcf99c1ce"
            }
        }
    }

    post {
        always{
            publishHTML([allowMissing: false,
                    alwaysLinkToLastBuild: false,
                    keepAll: true,
                    reportDir: '',
                    reportFiles: "${OUTPUT}.html",
                    reportName: "${LOGS}"
            ])       

            script {
                junit "${OUTPUT}.xml"
                emailext (
                    body:''' 
                        <p>Executed: <b>${JOB_NAME}:${BUILD_NUMBER}</b></p>
                        <p>View <a href="${BUILD_URL}/DevOps_20logs">DevOps logs</a></p>
                        <p>
                            =================================================================</br>
                            Total Run: ${TEST_COUNTS,var="total"}</br>
                            <font color="green">Passed: ${TEST_COUNTS,var="pass"}</font></br>
                            <font color="red">Failed: ${TEST_COUNTS,var="fail"}</font></br>
                            =================================================================
                        </p>''',
                    attachmentsPattern: "${OUTPUT}.html",
                    mimeType: "text/html",
                    subject: "Status: ${currentBuild.result?:'SUCCESS'} - \' ${JOB_NAME}:${BUILD_NUMBER}\'",
                    to: 'akhil.sureshbabu@innovapost.com'
                )
            }
        }
    }
}

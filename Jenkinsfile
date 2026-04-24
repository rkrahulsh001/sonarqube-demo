pipeline {
    agent any

    environment {
        SONAR_PROJECT_KEY = 'sonarqube-demo'
        SONAR_HOST_URL    = 'http://192.168.49.2:31900'
        BRANCH_ENV        = "${env.GIT_BRANCH?.replaceAll('origin/', '') ?: 'main'}"
        RECIPIENTS        = 'rkrahulsh001@gmail.com'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                echo "Branch: ${BRANCH_ENV}"
            }
        }

        stage('Setup') {
            steps {
                sh '''
                    pip3 install pytest pytest-cov \
                        --break-system-packages -q
                '''
            }
        }

        stage('Unit Tests') {
            steps {
                dir('src') {
                    sh '''
                        python3 -m pytest test_calculator.py \
                            -v \
                            --junitxml=test-results.xml \
                            --cov=calculator \
                            --cov-report=xml:../coverage.xml \
                            --cov-report=term-missing
                    '''
                }
            }
            post {
                always {
                    junit allowEmptyResults: true,
                          testResults: 'src/test-results.xml'
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        sonar-scanner \
                            -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                            -Dsonar.projectName='SonarQube Demo' \
                            -Dsonar.sources=src \
                            -Dsonar.tests=src \
                            -Dsonar.test.inclusions=src/test_*.py \
                            -Dsonar.python.coverage.reportPaths=coverage.xml \
                            -Dsonar.python.version=3 \
                            -Dsonar.branch.name=${BRANCH_ENV} \
                            -Dsonar.host.url=${SONAR_HOST_URL}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Build Artifact') {
            steps {
                echo "Quality Gate PASSED!"
                sh 'echo "Build: $(date) | Branch: ${BRANCH_ENV}" > build-info.txt'
                archiveArtifacts artifacts: 'build-info.txt'
            }
        }
    }

    post {
        always {
            echo "Build: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            echo "Status: ${currentBuild.currentResult}"
        }
        success {
            slackSend(
                channel: '#jenkins-notifications-',
                color: 'good',
                message: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER} | Quality Gate PASSED | Branch: ${BRANCH_ENV}",
                tokenCredentialId: 'slack-token'
            )
            emailext(
                subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "<h2 style='color:green'>Quality Gate PASSED</h2><p>Branch: ${BRANCH_ENV}</p><p><a href='${env.BUILD_URL}'>View Build</a></p>",
                mimeType: 'text/html',
                to: RECIPIENTS
            )
        }
        failure {
            slackSend(
                channel: '#jenkins-notifications-',
                color: 'danger',
                message: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER} | Quality Gate FAILED | Branch: ${BRANCH_ENV}",
                tokenCredentialId: 'slack-token'
            )
            emailext(
                subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: "<h2 style='color:red'>Quality Gate FAILED</h2><p>Branch: ${BRANCH_ENV}</p><p><a href='${env.BUILD_URL}'>View Build</a></p>",
                mimeType: 'text/html',
                to: RECIPIENTS,
                attachLog: true
            )
        }
    }
}

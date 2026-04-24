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
                tokenCredentialId: 'slack-token',
                message: """✅ *SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}*
Job    : ${env.JOB_NAME}
Build  : #${env.BUILD_NUMBER}
Branch : ${BRANCH_ENV}
URL    : ${env.BUILD_URL}

*Quick Links:*
🔗 <${env.BUILD_URL}|View Build Page>
✅ <${env.BUILD_URL}testReport|Test Results>
📊 <${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}|SonarQube Report>
📋 <${env.BUILD_URL}console|Console Output>

Status : SUCCESS ✅"""
            )
            emailext(
                subject: "✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
<!DOCTYPE html><html><body style="font-family:Arial,sans-serif;padding:20px;">
<div style="background:#28a745;padding:15px;border-radius:8px;margin-bottom:20px;">
  <h2 style="color:white;margin:0;">✅ SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}</h2>
</div>
<table style="width:100%;border-collapse:collapse;">
  <tr style="background:#f2f2f2;">
    <td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Job</td>
    <td style="padding:8px;border:1px solid #ddd;">${env.JOB_NAME}</td></tr>
  <tr><td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Build</td>
    <td style="padding:8px;border:1px solid #ddd;">#${env.BUILD_NUMBER}</td></tr>
  <tr style="background:#f2f2f2;">
    <td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Branch</td>
    <td style="padding:8px;border:1px solid #ddd;">${BRANCH_ENV}</td></tr>
  <tr><td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Quality Gate</td>
    <td style="padding:8px;border:1px solid #ddd;color:green;font-weight:bold;">PASSED ✅</td></tr>
</table>
<div style="margin-top:15px;">
  <a href="${env.BUILD_URL}" style="background:#0066cc;color:white;padding:8px 15px;border-radius:4px;text-decoration:none;margin-right:10px;">View Build</a>
  <a href="${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}" style="background:#28a745;color:white;padding:8px 15px;border-radius:4px;text-decoration:none;">SonarQube Report</a>
</div>
</body></html>""",
                mimeType: 'text/html',
                to: RECIPIENTS
            )
        }

        failure {
            slackSend(
                channel: '#jenkins-notifications-',
                color: 'danger',
                tokenCredentialId: 'slack-token',
                message: """❌ *FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}*
Job    : ${env.JOB_NAME}
Build  : #${env.BUILD_NUMBER}
Branch : ${BRANCH_ENV}
URL    : ${env.BUILD_URL}

*Quick Links:*
🔗 <${env.BUILD_URL}|View Build Page>
📋 <${env.BUILD_URL}console|Console Output>
🔍 <${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}|SonarQube Issues>

Status : FAILED ❌"""
            )
            emailext(
                subject: "❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
<!DOCTYPE html><html><body style="font-family:Arial,sans-serif;padding:20px;">
<div style="background:#dc3545;padding:15px;border-radius:8px;margin-bottom:20px;">
  <h2 style="color:white;margin:0;">❌ FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}</h2>
</div>
<table style="width:100%;border-collapse:collapse;">
  <tr style="background:#f2f2f2;">
    <td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Job</td>
    <td style="padding:8px;border:1px solid #ddd;">${env.JOB_NAME}</td></tr>
  <tr><td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Build</td>
    <td style="padding:8px;border:1px solid #ddd;">#${env.BUILD_NUMBER}</td></tr>
  <tr style="background:#f2f2f2;">
    <td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Branch</td>
    <td style="padding:8px;border:1px solid #ddd;">${BRANCH_ENV}</td></tr>
  <tr><td style="padding:8px;border:1px solid #ddd;font-weight:bold;">Quality Gate</td>
    <td style="padding:8px;border:1px solid #ddd;color:red;font-weight:bold;">FAILED ❌</td></tr>
</table>
<div style="margin-top:15px;">
  <a href="${env.BUILD_URL}console" style="background:#dc3545;color:white;padding:8px 15px;border-radius:4px;text-decoration:none;margin-right:10px;">Console Output</a>
  <a href="${SONAR_HOST_URL}/dashboard?id=${SONAR_PROJECT_KEY}" style="background:#ffc107;color:black;padding:8px 15px;border-radius:4px;text-decoration:none;">View Issues</a>
</div>
</body></html>""",
                mimeType: 'text/html',
                to: RECIPIENTS,
                attachLog: true
            )
        }
    }
}

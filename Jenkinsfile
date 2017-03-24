#!groovy
pipeline {
    agent any
    environment {
        IBM_CLOUD_DEVOPS_CREDS = credentials('xunrong_BM_CRED')
        IBM_CLOUD_DEVOPS_ORG = 'lix@us.ibm.com'
        IBM_CLOUD_DEVOPS_APP_NAME = 'Weather-V1-Xunrong'
        IBM_CLOUD_DEVOPS_TOOLCHAIN_ID = '1320cec1-daaa-4b63-bf06-7001364865d2'
        IBM_CLOUD_DEVOPS_WEBHOOK_URL = ''
    }
    tools {
        nodejs 'recent'
    }
    stages {
        stage('Build') {
            environment {
                GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
            }
            steps {
                checkout scm

                sh 'npm --version'
                sh 'npm install'
                sh 'grunt dev-setup --no-color'
            }
            post {
                success {
                    publishBuildRecord gitBranch: "master", gitCommit: "${GIT_COMMIT}", gitRepo: "https://github.com/xunrongl/DemoDRA-1", result:"SUCCESS"
                }
            }
        }
        stage('Unit Test and Code Coverage') {
            steps {
                sh 'grunt dev-test-cov --no-color -f'
            }
            post {
                always {
                    publishTestResult type:'unittest', fileLocation: './mochatest.json'
                    publishTestResult type:'code', fileLocation: './tests/coverage/reports/coverage-summary.json'
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
                echo "deploying to staging"
                sh '''
                        echo "CF Login..."
                        cf api $CF_API
                        cf login -u $CF_CREDS_USR -p $CF_CREDS_PSW -o $CF_ORG -s $CF_SPACE

                        echo "Deploying...."
                        export CF_APP_NAME="staging-$CF_APP"
                        cf delete $CF_APP_NAME -f
                        cf push $CF_APP_NAME -n $CF_APP_NAME -m 64M -i 1
                        export APP_URL=http://$(cf app $CF_APP_NAME | grep urls: | awk "{print $2}")
                    '''
            }
            post {
                success {
                    publishDeployRecord environment: "STAGING", appUrl: "google.com", result:"SUCCESS"
                }
            }
        }
        stage('FVT') {
            steps {
                sh 'grunt fvt-test --no-color -f'
            }
            post {
                always {
                    publishTestResult type:'fvt', fileLocation: './mochafvt.json', environment: 'STAGING'

                }
            }
        }
        stage('Gate') {
            steps {
                evaluateGate policy: 'Weather App Policy', forceDecision: 'true'
            }
        }
        stage('Deploy to Prod') {
            steps {
                sh 'echo "Deploy to Prod"'
            }
        }
    }
}
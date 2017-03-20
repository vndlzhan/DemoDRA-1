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

            steps {
                checkout scm
                try {
                	def GIT_COMMIT = sh(returnStdout: true, script: 'git rev-parse HEAD').trim()
                	sh 'npm --version'
                	sh 'npm install'
                	sh 'grunt dev-setup --no-color'
                	currentBuild.result = 'SUCCESS'
                }
                catch(Exception e) {
                	echo "build failed"
                	currentBuild.result = 'FAILURE'
                }
            }
            post {
            	always {
            		publishBuild gitBranch: "master", gitCommit: "${GIT_COMMIT}", gitRepo: "https://github.com/xunrongl/DemoDRA-1", result:"${currentBuild.result}"
            	}
            }
        }
        stage('Unit Test and Code Coverage') {
            steps {
                sh 'grunt fvt-test --no-color -f'
            }
            post {
                always {
                    // junit '**/target/*.xml'
                    sh 'Test completed'
                }
            }
        }
        stage('Deploy to Staging') {
            steps {
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
        }
        stage('FVT') {
            steps {
            	sh 'grunt fvt-test --no-color -f'
            }
        }
        stage('Deploy to Prod') {
	        steps {
	            sh 'echo "Deploy to Prod"'
	        }
	    }
    }
}
#!groovy
pipeline {
	agent any
	environment {
		CF_CREDS = credentials('xunrong_BM_CRED')
		CF_API = 'https://api.stage1.ng.bluemix.net'
		CF_ORG = 'lix@us.ibm.com'
		CF_SPACE = 'staging'
		CF_APP = 'Weather-V1-Xunrong'
	}
	tools {
		nodejs 'recent'
	}
    stages {
        stage('Build') {
            steps {
                checkout scm
                sh 'npm --version'
                sh 'npm install'
                sh 'grunt dev-setup --no-color '
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
pipeline {
	agent any
	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		}

		stage('Build Docker Image') {
			steps {
					sh '''
						docker build -t malassaad/capstone-blue .
					'''
				}
		}


		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([usernamePassword( credentialsId: 'dockerhub', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
					sh '''
						docker login -u ${USERNAME} -p ${PASSWORD}
						docker push malassaad/capstone-blue
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl config use-context arn:aws:eks:us-west-2:874895846642:cluster/capstonecluster
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}


		stage('Create the service in the cluster, redirect to blue') {
			steps {
				withAWS(region:'us-west-2', credentials:'ecr_credentials') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

	}
}

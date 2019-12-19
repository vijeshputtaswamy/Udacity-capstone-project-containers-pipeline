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
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker build -t vijeshputtaswamy/capstoneprojectaws .
					'''
				}
			}
		}

		stage('Push Image To Dockerhub') {
			steps {
				withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'Docker', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
					sh '''
						docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
						docker push vijeshputtaswamy/capstoneprojectaws
					'''
				}
			}
		}

		stage('Set current kubectl context') {
			steps {
				withAWS(region:'ap-southeast-2', credentials:'aws-capstone') {
					sh '''
						kubectl config use-context arn:aws:eks:ap-southeast-2:592414514106:cluster/capstonecluster1
					'''
				}
			}
		}

		stage('Deploy blue container') {
			steps {
				withAWS(region:'ap-southeast-2', credentials:'aws-capstone') {
					sh '''
						kubectl apply -f ./blue-controller.json
					'''
				}
			}
		}

		stage('Deploy green container') {
			steps {
				withAWS(region:'ap-southeast-2', credentials:'aws-capstone') {
					sh '''
						kubectl apply -f ./green-controller.json
					'''
				}
			}
		}

		stage('Create the service in the cluster and redirect to blue') {
			steps {
				withAWS(region:'ap-southeast-2', credentials:'aws-capstone') {
					sh '''
						kubectl apply -f ./blue-service.json
					'''
				}
			}
		}

		stage('Wait for user approval / Sanity check') {
            steps {
                input "Ready to redirect traffic to green?"
            }
        }

		stage('Create service in the cluster and redirect to green') {
			steps {
				withAWS(region:'ap-southeast-2', credentials:'aws-capstone') {
					sh '''
						kubectl apply -f ./green-service.json
					'''
				}
			}
		}

	}
}

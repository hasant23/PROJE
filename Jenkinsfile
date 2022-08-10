pipeline{

	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials('dockerhub')
	}

	stages {

		stage('Build') {

			steps {
				sh 'docker build -t hasant23/hello-python:latest .'
			}
		}
	}
	stage('SonarQube analysis') {
		steps {
			withSonarQubeEnv('sonarqube-container') {
				// Optionally use a Maven environment you've configured already
				 withMaven(maven:'maven') {
					sh 'mvn clean install -Dproject.name=${commitHash} sonar:sonar'
				}
			}
		}
	}
	stage("Quality Gate") {
		steps {
			waitForQualityGate abortPipeline: true
		}
	}

		stage('Login') {

			steps {
				sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
			}
		}


		stage('Push') {

			steps {
				sh 'docker push hasant23/flask-app:1.0'
			}
		}


		stage('Deploy to K8s')
		{
			steps{
				sshagent(['k8s-jenkins'])
				{
					sh 'scp -r -o StrictHostKeyChecking=no node-deployment.yaml username@102.10.16.23:/path'
					
					script{
						try{
							sh 'ssh username@102.10.16.23 kubectl apply -f /path/node-deployment.yaml --kubeconfig=/path/kube.yaml'

							}catch(error)
							

							}
					}
				}
			}
		}
	}

	post {
		always {
			sh 'docker logout'
		}
	}

}
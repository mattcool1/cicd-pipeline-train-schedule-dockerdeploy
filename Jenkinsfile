pipeline {
   environment {
     imagename = "mattcool1/train-schedule"
     registryCredential = 'docker_hub_login'
     app = ''
    }
    agent {label 'worker3'}
    stages {
        stage('Build') {
            steps {
                echo 'Running build automation'
                sh './gradlew build --no-daemon'
                archiveArtifacts artifacts: 'dist/trainSchedule.zip'
            }
        }
        stage('Build Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running docker image creation!'
                script {
                    app = docker.build imagename
                    app.inside {
                        sh 'echo $(curl localhost:8080)'
					}
                }
            }
        }
        stage('Push Docker Image') {
            when {
                branch 'master'
            }
            steps {
                echo 'Running docker image push!'
                script {
                    docker.withRegistry('', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
        stage ('DeployToProduction') {
			when {
				branch 'master'
			}
			steps {
				input 'Deploy to Production'
				milestone(1)
				withCredentials ([usernamePassword(credentialsId: 'webserver_login', usernameVariable: 'USERNAME', passwordVariable: 'USERPASS')]) {
					script {
						sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker pull mattcool1/train-schedule:${env.BUILD_NUMBER}\""
						try {
						   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker stop train-schedule\""
						   sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker rm train-schedule\""
						} catch (err) {
							echo: 'caught error: $err'
						}
						sh "sshpass -p '$USERPASS' -v ssh -o StrictHostKeyChecking=no $USERNAME@${env.prod_ip} \"docker run --restart always --name train-schedule -p 8080:8080 -d mattcool1/train-schedule:${env.BUILD_NUMBER}\""
					}
				}
			}
		}
    }
}

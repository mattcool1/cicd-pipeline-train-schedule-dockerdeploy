pipeline {
   environment {
     imagename = "mattcool1/train-schedule"
     registryCredential = 'docker_hub_login'
     app = ''
    }
    agent {label 'worker1'}
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
                    docker.withRegistry('https://registry.hub.docker.com', 'docker_hub_login') {
                        app.push("${env.BUILD_NUMBER}")
                        app.push("latest")
                    }
                }
            }
        }
    }
}

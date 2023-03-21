pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    } 
    stages{
        stage("sonar quality check"){
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonar-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                        sh 'java -version'
                    }
                    sleep(60)
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
        }
        stage("Docker build and docker push"){
            steps{
                script{
                    withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                        sh '''
                            docker build -t harmansingh9765/springbuild:${VERSION} .
                            docker login -u harmansingh9765 -p $docker_password
                            docker push harmansingh9765/springbuild:${VERSION}
                            docker rmi harmansingh9765/springbuild:${VERSION}
                        '''
                    }
                }
            }
        }
        stage("Identifying misconfigs with datree and helm"){
            steps{
                script{
                    dir('kubernetes/') {
                        withEnv(['DATREE_TOKEN=7830826c-6cba-4214-a269-426e7e94e25d']) {      
                        sh 'helm datree test myapp/'
                        }
                    }
                }
            }
        }

    }
    post {
		    always {
			    mail bcc: '', body: "<br>Project: ${env.JOB_NAME} <br>Build Number: ${env.BUILD_NUMBER} <br> URL de build: ${env.BUILD_URL}", cc: '', charset: 'UTF-8', from: '', mimeType: 'text/html', replyTo: '', subject: "${currentBuild.result} CI: Project name -> ${env.JOB_NAME}", to: "maincoding1997@gmail.com";  
		    }
	    }
}

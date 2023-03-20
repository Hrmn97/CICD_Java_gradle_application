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
    }
}

pipeline{
    agent any 
    stages{
        stage("sonar quality check"){
            agent {
                docker{
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'Sonar-access') {
                        sh 'chmodc+x gradlew'
                        sh './gradlew sonarqube'
                    }
                }
            }
        }
    }
}

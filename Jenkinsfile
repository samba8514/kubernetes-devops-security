pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }  
        stage('mvn test') {
            steps {
              sh "mvn test"
            }
            post {
              always {
                junit 'target/surefire-reports/*.xml'
                jacoco execPattern: 'target/jacoco.exec'
              }
            }
        }
        stage('Docker build and push') {
                steps {
                  sh "printenv"
                  sh 'docker build -t samba/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push samba/numeric-app:""$GIT_COMMIT""'
                }
            }  
    }
}

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
        stage('Mutation Tests - PIT') {
           steps {
             sh "mvn org.pitest:pitest-maven:mutationCoverage"
           }
           post {
             always{
               pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
              }
          }    
        }
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                 usernameVariable: 'DOCKER_USERNAME', 
                                                 passwordVariable: 'DOCKER_PASSWORD')]) {
                    sh 'echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin'
                }
            }
        }
        stage('Docker build and push') {
                steps {
                  sh "printenv"
                  sh 'docker build -t samba8514/numeric-app:""$GIT_COMMIT"" .'
                  sh 'docker push samba8514/numeric-app:""$GIT_COMMIT""'
                }
            } 
        stage('List pods') {
          steps{
              withKubeConfig([credentialsId: 'kubeconfig']) {
              sh 'kubectl get pods'
              }      
          }
      }
      stage('K8s deployment - DEV') {
          steps{
              withKubeConfig([credentialsId: 'kubeconfig']) {
              sh "sed -i 's#replace#samba8514/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"  
              sh 'kubectl apply -f k8s_deployment_service.yaml'
              }      
          }
      }    
    }
}

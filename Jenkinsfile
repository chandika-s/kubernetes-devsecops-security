pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }   
      stage('Unit tests - Junit and Jacoco') {
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
      stage('Mutation tests  - PIT') {
            steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
              }
            post {
              always {
                pimutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
            }
        }
      stage('SonarQube Analysis') {
        steps {
            sh "mvn sonar:sonar -Dsonar.projectKey=numeric-app -Dsonar.projectName='numeric-app' -Dsonar.host.url=http://3.111.41.229:9000 -Dsonar.login=sqb_c1a5a80b32dd6bf2841b5b2d6e5da90561e8f30d"
                }
              }
      stage('Docker build and push') {
            steps {
              withDockerRegistry([credentialsId: "dockerhub", url: ""]) {
                sh 'printenv'
                sh 'docker build -t chandikas/numbericapp:""$GIT_COMMIT"" .'
                sh 'docker push chandikas/numbericapp:""$GIT_COMMIT""'
                }                
              }
            }
      stage('Kubernetes Deployment - Dev') {
            steps {
              withKubeConfig([credentialsId: 'kubeconfig']) {
                sh "sed -i 's#replace#chandikas/numbericapp:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                sh "kubectl apply -f k8s_deployment_service.yaml"
                }                
              }
            }
    }
}
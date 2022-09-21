pipeline {
  agent any

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar'
            }
        }
        stage('Unit Tests - JUnit and Jacoco') {
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
        stage('Docker Build and Push') {
          steps {
            withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
              sh 'printenv'
              sh 'docker build -t rahul28/numeric-app:""$GIT_COMMIT"" .'
              sh 'docker push rahul28/numeric-app:""$GIT_COMMIT""'
            }
          }
        }
        stage('Mutation Tests - PIT') {
              steps {
                sh "mvn org.pitest:pitest-maven:mutationCoverage"
              }
              post {
                always {
                  pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
                }
              }
            }
        stage('SonarQube - SAST') {
              steps {
              withCredentials([string(credentialsId: 'sonarQube', variable: 'key')]){
                sh "mvn sonar:sonar \
                      -Dsonar.projectKey=numeric-application \
                      -Dsonar.host.url=http://devsecops-demo.northeurope.cloudapp.azure.com:9000 \
                      -Dsonar.login=${key}"
              }
              withSonarQubeEnv('SonarQube'){
                 timeout(time: 2, unit: 'MINUTES') {
                           script {
                             waitForQualityGate abortPipeline: true
                           }
                         }
                }
            }
           }
        stage('Kubernetes Deployment - DEV') {
              steps {
                withKubeConfig([credentialsId: 'kubeconfig']) {
                  sh "sed -i 's#replace#rahul28/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
                  sh "kubectl apply -f k8s_deployment_service.yaml"
                }
              }
            }
    }
}
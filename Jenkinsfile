  COLOR_MAP = [
    'SUCCESS': 'good',
    'FAILURE': 'danger',
]

pipeline {
      agent {
        label 'agent'
      }
   tools {
     maven "maven3.9.6"
   }

   stages {
     stage ("git clone") {
       steps {
         git branch: 'main', url: 'https://github.com/Ibrahim-Sahabi/web-app.git'
       }  
     }

      stage ("Build with Maven") {
       steps {
         sh "mvn clean"
       }  
     }

      stage ("Test with Maven") {
       steps {
         sh "mvn test"
       }  
     }

      stage ("Package with Maven") {
       steps {
         sh "mvn package"
       }  
     }

      stage('SonarQube Analysis') {
          environment {
              ScannerHome = tool 'Sonar6.1'
          }
             steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh "${ScannerHome}/bin/sonar-scanner -Dsonar.projectKey=jomacs"
                    }
                }
             }
        }

        stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
            }
        }

        stage ("Upload to Nexus") {
         steps {
           nexusArtifactUploader artifacts: [[artifactId: 'maven-web-application', classifier: '', file: '/home/ubuntu/agent_home/workspace/webapp-pipeline-jenkinsfile/target/web-app.war', type: 'war']], credentialsId: 'nexus-id', groupId: 'com.mt', nexusUrl: '3.149.245.152:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'webapp-release', version: '3.9.6-RELEASE'
          }
        }

        stage ("Deploy to UAT") {
          steps {
            deploy adapters: [tomcat9(credentialsId: 'tomcat-cred2', path: '', url: 'http://3.137.216.136:8080')], contextPath: null, war: 'target/*.war'
          }
        }
    }

    post {
        always {
            slackSend channel: 'team-usa', 
                      color: COLOR_MAP[currentBuild.currentResult],
                      message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER}"
        }
    }
}




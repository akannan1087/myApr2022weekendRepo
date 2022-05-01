pipeline {
    agent any
    
    tools {
        maven "Maven3"
    }

    stages {
        stage('Checkoout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: '39a0281c-5ab6-4158-bb33-b7b1aab87117', url: 'https://github.com/akannan1087/myApr2022weekendRepo']]])
            }
        }
        
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
        }

        stage ("Code scan") {
            steps {
                withSonarQubeEnv ("SonarQube") {
                sh "mvn sonar:sonar -f MyWebApp/pom.xml"
            }
          }
        }
        
        stage("Quality Gate") {
          steps{
          timeout(time: 1, unit: 'HOURS') {
             waitForQualityGate abortPipeline: true
            }
          } 
        }
        stage ("Nexus upload") {
            steps{
            nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '5f169013-6b16-4418-9b86-443c2eb41581', groupId: 'com.dept.app', nexusUrl: 'ec2-3-137-200-230.us-east-2.compute.amazonaws.com:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
          } 
        }

        stage ("dev deploy") {
            steps{
                deploy adapters: [tomcat9(credentialsId: '5ecba445-25fe-4ed7-bf36-9d15cec78cd3', path: '', url: 'http://ec2-18-188-253-180.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
            }
        }
    
        stage ("slack") {
            steps {
                slackSend channel: 'apr-2022-weekend-batch', message: 'DEV Deployment was successful.'
            }
        }

   stage ('DEV Approve')  {
       steps {
        echo "Taking approval from Manager for QA Deploy"     
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you approve to deploy into QA environment?', submitter: 'devmanager@email.com'
        }
       }
     }
     
    stage ("QA deploy") {
        steps {
        deploy adapters: [tomcat9(credentialsId: '5ecba445-25fe-4ed7-bf36-9d15cec78cd3', path: '', url: 'http://ec2-18-188-253-180.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
     }
    }

    stage ("QA notify") {
        steps {
            slackSend channel: 'apr-2022-weekend-batch, qa-testing-team', message: 'QA Deployment was successful. Please start QA testing..'
          }
        }

   stage ('QA Approve')  {
       steps {
        echo "Taking approval from Manager for PROD Deploy"     
        timeout(time: 3, unit: 'DAYS') {
        input message: 'Do you approve to deploy into PROD environment?', submitter: 'admin'
        }
     }
   }
     
    stage ("PROD deploy") {
        steps {
        deploy adapters: [tomcat9(credentialsId: '5ecba445-25fe-4ed7-bf36-9d15cec78cd3', path: '', url: 'http://ec2-18-188-253-180.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
      }
    }

    stage ("Final notify") {
        steps {
        slackSend channel: 'apr-2022-weekend-batch, product-owners-teams', message: 'PROD Deployment was successful. Please start final testing and inform customer..'
     }
    }
    }
}

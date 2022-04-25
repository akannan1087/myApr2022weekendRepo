pipeline {
    agent any
    tools {
        maven "Maven3"
    }
    
    stages {
        stage('Checkout') {
            steps {
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'c5d1add2-cbcb-470b-88f0-81b0076f9d93', url: 'https://github.com/akannan1087/myMar2022WeekdayBatchRepo']]])

            }
        }
        
        stage ("Build") {
            steps {
                sh "mvn clean install -f MyWebApp/pom.xml"
            }
            
        }

        stage ("Code scan") {
            steps {
                withSonarQubeEnv("Sonarqube") {
                sh "mvn sonar:sonar -f MyWebApp/pom.xml"
            }
         }
        }
        
        stage ("Nexus upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '03405c4b-8bfd-42f6-94b5-2fcb1657b7c0', groupId: 'com.dept.app', nexusUrl: 'ec2-3-140-187-14.us-east-2.compute.amazonaws.com:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'

            }
        }
        
        stage ("DEV deploy") {
            steps {
                 deploy adapters: [tomcat9(credentialsId: '3fc8c614-a0c1-4fae-a0ce-af884cccf5d1', path: '', url: 'http://ec2-3-135-116-26.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'   

            }
        }
        
        stage ("Slack") {
            steps {
                slackSend channel: 'mar-2022-weekday-batch,feb-2022-weekday-batch', message: 'DEV Deployment was successful, please start DEV testing'
            }
        }


    stage ("DEV Approve") {
        steps {
        echo "Taking approval from DEV Manager for QA Deploy"     
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you approve to deploy into QA environment?', submitter: 'admin'
        }
    }
  }
    stage ("QA deploy") {
        steps {
            deploy adapters: [tomcat9(credentialsId: '3fc8c614-a0c1-4fae-a0ce-af884cccf5d1', path: '', url: 'http://ec2-3-135-116-26.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'   
        }
    }
    stage ("QA Notify") {
        steps {
            slackSend channel: 'mar-2022-weekday-batch,feb-2022-weekday-batch', message: 'QA Deployment was successful, please start your functional testing'
        }
    }
    stage ("QA Approve") {
        steps {
            echo "Taking approval from QA Manager for PROD Deploy"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve to deploy into PROD environment?', submitter: 'admin'
          }
        }
    }

    stage ("PROD deploy") {
    steps {
        deploy adapters: [tomcat9(credentialsId: '3fc8c614-a0c1-4fae-a0ce-af884cccf5d1', path: '', url: 'http://ec2-3-135-116-26.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'   
        }
    }
    stage ("final Notify") {
        steps {
            slackSend channel: 'product-owners-teams', message: 'PROD Deployment was successful, please inform to the end customers..'
       }
     }
    }
}

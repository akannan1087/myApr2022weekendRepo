pipeline {
    agent any
    tools {
        maven "Maven3"
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'a07bf9c3-cc44-48d4-b759-4e7a8cb9cf68', url: 'https://github.com/akannan1087/myMay2022WeekendBatch']]])
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
                sh "mvn clean install -f MyWebApp/pom.xml"
                }
            }
        }
        
        stage ("Nexus upload") {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '23ddd6e6-a246-475b-87a9-0992b8175483', groupId: 'com.dept.app', nexusUrl: 'ec2-54-196-178-159.compute-1.amazonaws.com:8081/', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT' 
            }
        }
        
    stage ("DEV deploy") {
        steps {
         deploy adapters: [tomcat9(credentialsId: '148dbd49-8449-4716-98ca-5ae47a0962af', path: '', url: 'http://ec2-18-217-46-200.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
        }
            
        }
    
    stage ("Slack Notify") {
        steps {
       slackSend channel: 'apr-2022-weekend-batch', message: 'DEV Deployment was successful, please start DEV testing before QA deploy' 
      }
    }
    stage ("DEV approve") {
        steps {
        echo "Taking approval from DEV Manager for QA Deploy"     
        timeout(time: 7, unit: 'DAYS') {
        input message: 'Do you approve to deploy into QA environment?', submitter: 'admin'
        }
    }
}
    stage ("QA deploy") {
        steps {
         deploy adapters: [tomcat9(credentialsId: '148dbd49-8449-4716-98ca-5ae47a0962af', path: '', url: 'http://ec2-18-217-46-200.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
        }
            
        }

    stage ("QA Notify") {
        steps {
         slackSend channel: 'apr-2022-weekend-batch,qa-testing-team', message: 'QA Deployment was successful, please start QA testing before UAT deploy' 
     }
    }
     stage ("QA approve") {
        steps {
            echo "Taking approval from QA Manager for PROD Deploy"     
            timeout(time: 7, unit: 'DAYS') {
            input message: 'Do you approve to deploy into PROd environment?', submitter: 'admin'
        }
      }
    }

    stage ("PROD deploy") {
        steps {
         deploy adapters: [tomcat9(credentialsId: '148dbd49-8449-4716-98ca-5ae47a0962af', path: '', url: 'http://ec2-18-217-46-200.us-east-2.compute.amazonaws.com:8080/')], contextPath: null, war: '**/*.war'
      }
    }

    stage ("Final Notify") {
        steps {
        slackSend channel: 'product-owners-teams', message: 'PROD Deployment was successful, please start QA testing before UAT deploy' 
      }
    }
    }
}

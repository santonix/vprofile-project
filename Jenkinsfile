

def colorMap = [
    SUCCESS: 'good',
    FAILURE: 'danger',
    UNSTABLE: 'warning'
]    

pipeline {
    agent any
    tools {
        jdk 'jdk17'
        maven 'maven3'

    }

    environment {
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUS_URL = "34.134.153.215:8081"
        NEXUS_REPOSITORY = "vprofile-release"
	    NEXUS_REPOGRP_ID    = "vprofile-grp-repo"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
        NEXUSIP = "34.134.153.215"
        NEXUSPORT = "8081" 
        NEXUS_LOGIN = "nexuslogin"
        RELEASE_REPO = "vprofile-release"
        COLOR_MAP = '{"success": "green", "failure": "red"}' // Define COLOR_MAP in the environment

    }

    stages {

        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('CODE ANALYSIS with SONARQUBE') {
          
		  environment {
             scannerHome = tool 'sonar-scanner'
          }

          steps {
            withSonarQubeEnv('sonar-server') {
               sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
            }
          }
        }    

        stage("Quality Gate") {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }


        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${RELEASE_REPO}",
                  credentialsId: "${NEXUS_LOGIN}",
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }


    }

    post {
        always {
            echo 'Slack Notifications.'
            
            script {
                def color = colorMap[currentBuild.currentResult.toUpperCase()] ?: 'warning'
                
                slackSend channel: '#jenkinscicd',
                    color: color,
                    message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \nMore info at: ${env.BUILD_URL}",
                    tokenCredentialId: SLACK_CREDENTIALS_ID
            }
        }
    }
}
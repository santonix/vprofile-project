pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "JDK8"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'walter'
        RELEASE_REPO = 'vprofile-release'
        CENTRAL_REPO = 'vpro-maven-central'
        NEXUSIP = '172.31.31.161'
        NEXUSPORT = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN = 'nexus-cred'
        SONARSERVER = 'sonar-server'
        SONARSCANNER = 'sonar-scanner'
        SONAR_METADATA_FILE_PATH = "${env.WORKSPACE}/.scannerwork/report-task.txt"
    }

    stages {
        stage('build'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }
            post {
                success {
                    echo "Now Archiving..."
                    archiveArtifacts artifacts: "**/*.war"
                }
            }
        }

        stage('test') {
            steps {
                sh 'mvn -s settings.xml test'
            }

        }

        stage('checkstyle analysis') {
            steps {
                sh 'mvn  -s settings.xml  checkstyle:checkstyle'
            }
        }

        stage('static code analysis') {
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
                    sh '''${SONARSCANNER}/bin/sonar-scanner \
                    -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/main/java \
                    -Dsonar.tests=src/test/java \
                    -Dsonar.java.binaries=target/classes \
                    -Dsonar.java.test.binaries=target/test-classes \
                    -Dsonar.scanner.metadataFilePath=${SONAR_METADATA_FILE_PATH} \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml \
                    -Dsonar.sourceEncoding=UTF-8'''
                }
            }
        }
    }
}

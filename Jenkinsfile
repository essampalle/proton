pipeline {
    agent any

    tools {
        maven "MAVEN3.9.9"
        jdk "JDK-21"
    }

    environment {
        SNAP_REPO     = 'vprofile-snapshot'
        NEXUS_USER    = 'admin'
        NEXUS_PASS    = 'Harsha@6300'
        RELEASE_REPO  = 'vprofile-release'
        CENTRAL_REPO  = 'vpro-maven-central'
        NEXUSIP       = '172.31.7.158'
        NEXUSPORT     = '8081'
        NEXUS_GRP_REPO = 'vpro-maven-group'
        NEXUS_LOGIN   = 'nexuslogin'
        SONAR_SERVER   = 'sonarserver'
        SONAR_SCANNER   = 'sonarscanner'


    }

    stages {

        stage('Build') {
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
            }

            post {
                success {
                    echo "Now Archiving"
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
        }

        stage('Test') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }

        stage('Checkstyle Analysis') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
      
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONAR_SCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONAR_SERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
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
    
    }
}
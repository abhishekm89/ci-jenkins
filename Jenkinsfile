pipeline {
    agent any
    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }
    environment {
        SNAP_REPO = "vprofile-repo"             // Nexus Maven(Hosted) Snapshot Repository
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin123"
        RELEASE_REPO = "vprofile-release"       // Nexus Maven(Hosted) Release Repository
        CENTRAL_REPO = "vpro-maven-central"     // Nexus Maven(Proxy) Repository
        NEXUSIP = "172.31.50.80"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vpro-maven-group"     // Nexus Maven(Group) Repository 
        NEXUS_LOGIN = "nexuslogin"              // Jenkins Global Credentials ID
        SONAR_SERVER = "sonarserver"
        SONAR_SCANNER = "sonarscanner"
    }
    stages {
        stage ('BUILD') {
            steps {
                sh 'mvn -s settings.xml' 
            }
            post {
                success {
                    echo " *** Archiving Artifact ***"
                    archiveArtifact artifacts: '**/*.war'
                }
            }
        }
        stage ('UNIT TEST') {
            steps {
                sh 'mvn -s settings.xml test'
            }
        }
        stage ('CHECKSTYLE ANALYSIS') {
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
        }
        stage('SONARQUBE - CODE ANALYSIS') {
            environment {
                scannerHome = tool "${SONAR_SCANNER}"
            }
            steps {
                withSonarQubeEnv('sonar-pro') {
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
    }
}
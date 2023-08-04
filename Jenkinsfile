def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
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
        NEXUSIP = "172.31.80.28"
        NEXUSPORT = "8081"
        NEXUS_GRP_REPO = "vpro-maven-group"     // Nexus Maven(Group) Repository 
        NEXUS_LOGIN = "nexuslogin"              // Jenkins Global Credentials ID
        SONARSERVER = "sonarserver"
        SONARSCANNER = "sonarscanner"
    }
    stages {
        stage ('BUILD') {
            steps {
                sh 'mvn install -s settings.xml' 
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
        stage ('SONARQUBE - CODE ANALYSIS') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
                withSonarQubeEnv("${SONARSERVER}") {
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
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}

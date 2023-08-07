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
        ARTIFACT_NAME = "vprofile-v${BUILD_ID}.war"
        AWS_S3_BUCKET = 'vprofile-cicd-beanstalk'       // S3 Bucket Name
        AWS_EB_APP_NAME = 'vproapp'                     // ElasticBeanstalk Application Name
        AWS_EB_ENVIRONMENT = 'Vproapp-env'              // ElasticBeanstalk Environment Name
        AWS_EB_APP_VERSION = "${BUILD_ID}"
    }
    stages {
        stage ('BUILD') {
            steps {
                sh 'mvn -s settings.xml install' 
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
        stage ('QUALITY GATE') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Parameter indicates whether to set pipeline to UNSTABLE if Quality Gate fails
                    // true = set pipeline to UNSTABLE, false = don't
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage ('UPLOAD ARTIFACT') {
            steps {
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
        stage ('DEPLOY TO BEANSTALK STAGING ENVIRONMENT') {
            steps {
                withAWS(credentials: 'awsbeancreds', region: 'us-east-1') {
                    sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                    // uploads artifacts to S3 Bucket
                    sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                    // creates new application version in Beanstalk application
                    sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
                    // deploys application version into Beanstalk environment
                }
            }
        }
    }
    post {
        always {
            echo 'Slack Notifications.'
            slackSend channel: '#jenkinscicd',
                color: COLOR_MAP[currentBuild.currentResult],
                message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME}-build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
        }
    }
}

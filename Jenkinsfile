// def buildNumber = Jenkins.instance.getItem('<Jenking-Staging-Job-Name>').lastSuccessfulBuild.number
def buildNumber = Jenkins.instance.getItem('cicd-jenkins-bean-stage').lastSuccessfulBuild.number

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
        ARTIFACT_NAME = "vprofile-v${buildNumber}.war"  // Fetch BUILD_ID version from Staging Pipeline
        AWS_S3_BUCKET = 'vprofile-cicd-beanstalk'       // S3 Bucket Name
        AWS_EB_APP_NAME = 'vproapp'                     // ElasticBeanstalk Application Name
        AWS_EB_ENVIRONMENT = 'Vproapp-prod-env'         // ElasticBeanstalk Production Environment Name
        AWS_EB_APP_VERSION = "vprofile-app-v${buildNumber}"
    }
    stages {
        stage ('DEPLOY TO BEANSTALK PRODUCTION ENVIRONMENT') {
            steps {
                withAWS(credentials: 'awsbeancreds', region: 'us-east-1') {
                    // sh 'aws s3 cp ./target/vprofile-v2.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME'
                    // Not Required. Artifact has been already upload by Staging Pipeline

                    // sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                    // Not Required. Staging Pipeline has already created new application version in Beanstalk application
                    
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

def COLOR_MAP = [
    'SUCCESS': 'good', 
    'FAILURE': 'danger',
]
pipeline {
    agent any
    environment {
        NEXUS_USER = "admin"
        NEXUS_PASS = "admin123"
        RELEASE_REPO = "vprofile-release"       // Nexus Maven(Hosted) Release Repository
        NEXUSIP = "172.31.80.28"
        NEXUSPASS = credentials('nexuspass')    // Secret Text stored in JenkinsCredentials (admin123)
    }                                           // Because we just need password. NEXUS_PASS -> uname_and_pass
    stages {
        stage ('SETUP PARAMETERS') {
            steps {
                script {
                    properties ([
                        Parameters ([
                            string (
                                defaultValue: '',
                                name: 'BUILD',
                            ),
                            string (
                                defaultValue: '',
                                name: 'TIME',
                            )
                        ])
                    ])
                }
            }
        }
        stage ('ANSIBLE DEPLOY TO PRODUCTION') {
            steps {
                ansiblePlaybook([
                inventory   : 'ansible/stage.inventory',
                playbook    : 'ansible/site.yml',
                installation: 'ansible',
                colorized   : true,
			    credentialsId: 'applogin-prod',      // AppServer(tomcat) EC2 AccessKeys saved in Jenkins Credentials
			    disableHostKeyChecking: true,
                extraVars   : [
                   	USER: "${NEXUS_USER}",
                    PASS: "${NEXUSPASS}",
			        nexusip: "${NEXUSIP}",               
			        reponame: "${RELEASE_REPO}",
			        groupid: "QA",
			        time: "${env.TIME}",            // "TIME" Variable dont exist. Need to take as argument from user
			        build: "${env.BUILD}",          // "BUILD" Variable dont exist. Need to take as argument from user
                    artifactid: "vproapp",
			        vprofile_version: "vproapp-${env.BUILD}-${env.TIME}.war"
                    ]
                ])

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

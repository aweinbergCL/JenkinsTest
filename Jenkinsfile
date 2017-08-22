#!groovy

def masterBuildPulseEndpoint = 'http://corelogic-project-monitor-production.cfapps.io/projects/REPLACETHIS/status'

if (env.BRANCH_NAME == 'master') {
    properties([[$class  : 'BuildDiscarderProperty',
                 strategy: [$class: 'LogRotator', artifactNumToKeepStr: '3', numToKeepStr: '3']],
                [$class   : 'HudsonNotificationProperty',
                 endpoints: [[event: 'all', format: 'JSON', loglines: 0, protocol: 'HTTP', timeout: 30000, url: masterBuildPulseEndpoint]]]])
} else {
    properties([[$class  : 'BuildDiscarderProperty',
                 strategy: [$class: 'LogRotator', artifactNumToKeepStr: '3', numToKeepStr: '3']]])
}

// IMPORTANT NOTE:
// Please do not put "stage" that waits in a node scope since it
// will occupy an executor during the entire wait period

node('linux') {

    stages {
        stage('Build') {
            steps {
                echo 'Building...'
                sh 'npm install'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing...'
                sh 'npm test'
            }
        }
    }

}

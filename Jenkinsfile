#!groovy

//def masterBuildPulseEndpoint = 'http://corelogic-project-monitor-production.cfapps.io/projects/REPLACETHIS/status'
//
//if (env.BRANCH_NAME == 'master') {
//    properties([[$class  : 'BuildDiscarderProperty',
//                 strategy: [$class: 'LogRotator', artifactNumToKeepStr: '3', numToKeepStr: '3']],
//                [$class   : 'HudsonNotificationProperty',
//                 endpoints: [[event: 'all', format: 'JSON', loglines: 0, protocol: 'HTTP', timeout: 30000, url: masterBuildPulseEndpoint]]]])
//} else {
//    properties([[$class  : 'BuildDiscarderProperty',
//                 strategy: [$class: 'LogRotator', artifactNumToKeepStr: '3', numToKeepStr: '3']]])
//}

// IMPORTANT NOTE: 
// Please do not put "stage" that waits in a node scope since it
// will occupy an executor during the entire wait period



    stage('Clean Workspace') {
        // Delete workspace
        echo "clean workspace is running"
        deleteDir()
    }

    stage('Checkout') {
        // Checkout Code
        checkout scm
    }

    stage('Build') {

        // Update build name
        getCommitRevision()
        currentBuild.displayName = "#" + env.BUILD_NUMBER + "-" + env.GITHASH.substring(0, 7)
        def localdisplayName = env.BUILD_NUMBER + "-" + env.GITHASH.substring(0, 7)
        // Build
        setJavaHomeOnPath()
        sh "./gradlew clean assemble -x test --console=plain --no-daemon --stacktrace --rerun-tasks"

        // Archive files
        step([$class: 'ArtifactArchiver', artifacts: 'build/**/*.jar, manifests/manifest*.yml', , fingerprint: true])

        // Stash artifacts for later use in this run
        stash includes: 'build/**/*.jar, manifests/manifest*.yml', name: 'artifacts'
    }



def setJavaHomeOnPath() {
    env.JAVA_HOME = "${tool 'Java8'}"
    env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    sh 'java -version'
}

def getCommitRevision() {
    sh 'git rev-parse HEAD > commit'
    env.GITHASH = readFile('commit').trim()
    sh "git config --local remote.origin.url | sed -n 's#.*/\\([^.]*\\)\\.git#\\1#p' > reponame"
    env.GITREPONAME = readFile('reponame').trim()
    sh 'env | sort'
}

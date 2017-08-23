#!groovy



    properties([[$class  : 'BuildDiscarderProperty',
                 strategy: [$class: 'LogRotator', artifactNumToKeepStr: '3', numToKeepStr: '3']]])

// IMPORTANT NOTE:
// Please do not put "stage" that waits in a node scope since it
// will occupy an executor during the entire wait period

node {

    stage('Clean Workspace') {
        // Delete workspace
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

}

stage('Build Checkpoint') {
    try {
        checkpoint 'Build Complete Checkpoint'
    } catch (NoSuchMethodError _) {
        echo 'Checkpoint feature available in Jenkins Enterprise by CloudBees.'
    }
}

node {
    stage 'Parallel Testing Steps'
    parallel(failFast: true,
            'js unit tests': {
                node {
                    // unstash files need to run test
                    sh 'echo js_unit_tests ; sleep 20'
                }
            },
            'java unit tests': {
                node {
                    // unstash files need to run test
                    sh "echo java_unit_tests ; sleep 4"
                }
            },
            'java integration tests': {
                node {
                    // unstash files need to run test
                    sh "echo java_int_tests ; sleep 15"
                }
            },
            'e2e tests': {
                node {
                    // unstash files need to run test
                    sh "echo e2e_tests ; sleep 15"
                }
            })
}

stage('Test Checkpoint') {
    try {
        checkpoint 'Test Complete Checkpoint'
    } catch (NoSuchMethodError _) {
        echo 'Checkpoint feature available in Jenkins Enterprise by CloudBees.'
    }
}



node {
    unstash 'everything'

    stage 'Snapshot'

    sh './gradlew assemble uploadArchives --console=plain --stacktrace'
    deleteDir()
}

node {
    unstash 'everything'

    stage 'Release'

    sh './gradlew release -Prelease.useAutomaticVersion=true -x test'
    deleteDir()
}


//
//stage('Upload Artifacts') {
//    node {
//
//
//
////        def utils = fileLoader.fromGit('./utils.groovy', 'https://github.com/corelogic/cd-pipeline.git', 'master', '774aeeb5-4206-427f-bdb8-33f22bde0252', '')
////        def filename = 'jenkins-test-pipeline.zip'
////        def localdisplayName = env.BUILD_NUMBER + "-" + env.GITHASH.substring(0, 7)
////        def buildinfo = readJSON text: "{}"
////
////
////        buildinfo['sha'] = env.GITHASH
////        buildinfo['branch'] = env.BRANCH_NAME
////        buildinfo['repo'] = env.GITREPONAME
////        buildinfo['buildnumber'] = localdisplayName
////        buildinfo['buildurl'] = env.BUILD_URL
////        unstash 'artifacts'
////        writeJSON file: 'buildinfo.json', json: buildinfo
////        zip zipFile: filename
////        utils.UploadToNexus(filename,localdisplayName)
//    }
//}

def setJavaHomeOnPath() {
    env.JAVA_HOME = "${tool 'jdk1.8.0_131'}"
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
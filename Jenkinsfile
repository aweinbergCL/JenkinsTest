#!groovy
commitRevision = ''

//credentialsId = ['SB'  : '7cca203b-c77b-4022-9e88-8ff79937c3d5']

node('linux') {
    deleteDir()
    properties([[$class  : 'BuildDiscarderProperty',
                     strategy: [$class: 'LogRotator', numToKeepStr: '30', artifactDaysToKeepStr: '14', artifactNumToKeepStr: '20']]])

    sh 'env | sort'

    stage 'Checkout'
    checkout scm

    stage 'Assemble'

    setJavaHomeOnPath()

    retry(2) {
        try {
            sh './gradlew build --console=plain --stacktrace -x test'
        } catch (Exception e) {
            throw e
        } finally {
            //step([$class: 'JUnitResultArchiver', testResults: 'build/test-results/**/*.xml', allowEmptyResults: true])
        }
    }

    stash name:'everything', includes:'**/*', useDefaultExcludes: false
    deleteDir()
}
checkpoint 'Build Complete'

//removed deploy to sandbox

node('linux') {
    unstash 'everything'

    stage 'Snapshot'

    sh './gradlew assemble uploadArchives --console=plain --stacktrace'
    deleteDir()
}
checkpoint 'Snapshot Deployed to Nexus'

stage 'Promotion'

timeout(time: 5, unit: 'DAYS') {
    input 'Deploy to nexus release?'
}

node('linux') {
    unstash 'everything'

    stage 'Release'

    sh './gradlew release -Prelease.useAutomaticVersion=false -x test --stacktrace --debug'
    deleteDir()
}

checkpoint 'Release Deployed to Nexus'

def setJavaHomeOnPath() {
    env.JAVA_HOME = "${tool 'Java8'}"
    env.PATH = "${env.JAVA_HOME}/bin:${env.PATH}"
    sh 'java -version'
}


#!groovy

pipeline {
    agent none
    stages {
        stage('Build') {
            agent any
            steps {
                checkout scm
                echo "building"
            }
        }
        stage('Test on Linux') {
            agent {
                label 'linux'
            }
            steps {
                unstash 'app'
                echo "test on Linux"
            }
            post {
                always {
                    echo "unit test"

                }
            }
        }
        stage('Test on Windows') {
            agent {
                label 'windows'
            }
            steps {
                unstash 'app'
                echo "testing on windows"
            }
            post {
                always {
                    echo "unit test 2"

                }
            }
        }
    }
}
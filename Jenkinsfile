//Jenkinsfile (Declarative Pipeline)
pipeline {
    agent {
        docker { image 'secondarynamenode' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}

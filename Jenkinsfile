pipeline {
    agent { label 'ubuntu-wsl' }

    stages {

        stage('Checkout') {
            steps {
                git(
                    credentialsId: 'github-cred',
                    url: 'https://github.com/sdineshgandhi/spring-petclinic-devops.git',
                    branch: 'main'
                )
            }
        }

        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t spring-petclinic:latest .'
            }
        }
    }
}

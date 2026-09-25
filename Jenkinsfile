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

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh '''
                        mvn sonar:sonar \
                          -Dsonar.projectKey=spring-petclinic \
                          -Dsonar.projectName=spring-petclinic
                    '''
                }
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t spring-petclinic:latest .'
            }
        }
    }
}

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

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --no-progress \
                      spring-petclinic:latest
                '''
            }
        }
          
         cat Jenkinsfile
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

        stage('Trivy Scan') {
            steps {
                sh '''
                    trivy image \
                      --severity HIGH,CRITICAL \
                      --no-progress \
                      spring-petclinic:latest
                '''
            }
        }

        stage('Docker Push to Nexus') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'nexus-cred',
                        usernameVariable: 'NEXUS_USER',
                        passwordVariable: 'NEXUS_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$NEXUS_PASSWORD" | docker login 172.22.100.88:8891 \
                            -u "$NEXUS_USER" \
                            --password-stdin

                        docker tag spring-petclinic:latest \
                            172.22.100.88:8891/petclinic-docker/spring-petclinic:${BUILD_NUMBER}

                        docker push \
                            172.22.100.88:8891/petclinic-docker/spring-petclinic:${BUILD_NUMBER}

                        docker logout 172.22.100.88:8891
                    '''
                }
            }
        }
    }
}


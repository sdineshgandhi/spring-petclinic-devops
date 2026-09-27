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

        stage('Docker Push to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-cred',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_TOKEN'
                    )
                ]) {
                    sh '''
                        echo "$DOCKERHUB_TOKEN" | docker login \
                            -u "$DOCKERHUB_USER" \
                            --password-stdin

                        docker tag spring-petclinic:latest \
                            "$DOCKERHUB_USER/spring-petclinic:${BUILD_NUMBER}"

                        docker push \
                            "$DOCKERHUB_USER/spring-petclinic:${BUILD_NUMBER}"

                        docker logout
                    '''
                }
            }
        }
         stage('Create Octopus Release') {
             steps {
                 withCredentials([
                     string(
                         credentialsId: 'octopus-api-key',
                         variable: 'OCTOPUS_API_KEY'
            )
        ]) {
            sh '''
                octopus release create \
                    --server "$OCTOPUS_URL" \
                    --apiKey "$OCTOPUS_API_KEY" \
                    --space "$OCTOPUS_SPACE" \
                    --project "$OCTOPUS_PROJECT" \
                    --releaseNumber "$BUILD_NUMBER"
            '''
        }
    }
}
    }
}

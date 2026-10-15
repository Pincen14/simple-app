
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'USERNAME_DOCKERHUB/simple-app'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                bat 'echo Mulai build aplikasi Flask'
            }
        }

        stage('Build Docker Image') {
            steps {
                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    powershell '''
                        $env:DH_TOKEN | docker login --username $env:DH_USER --password-stdin
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        $image = "$($env:IMAGE_NAME):$($env:BUILD_NUMBER)"

                        docker push $image
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        docker tag $image "$($env:IMAGE_NAME):latest"

                        docker push "$($env:IMAGE_NAME):latest"
                        if ($LASTEXITCODE -ne 0) { exit $LASTEXITCODE }

                        docker logout
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Selesai build'
        }
    }
}

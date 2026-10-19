
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'vincenttt14/simple-app'
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
                        $psi = New-Object System.Diagnostics.ProcessStartInfo
                        $psi.FileName = "docker.exe"
                        $psi.Arguments = "login --username $env:DH_USER --password-stdin"
                        $psi.UseShellExecute = $false
                        $psi.RedirectStandardInput = $true
                        $psi.RedirectStandardOutput = $true
                        $psi.RedirectStandardError = $true

                        $p = [System.Diagnostics.Process]::Start($psi)
                        $p.StandardInput.Write($env:DH_TOKEN)
                        $p.StandardInput.Close()

                        $output = $p.StandardOutput.ReadToEnd()
                        $errorOutput = $p.StandardError.ReadToEnd()
                        $p.WaitForExit()

                        if ($p.ExitCode -ne 0) {
                            throw "Docker login gagal: $errorOutput"
                        }

                        Write-Host "Docker login berhasil"

                        $image = "$($env:IMAGE_NAME):$($env:BUILD_NUMBER)"
                        $latest = "$($env:IMAGE_NAME):latest"

                        try {
                            docker push $image
                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker push gagal"
                            }

                            docker tag $image $latest
                            if ($LASTEXITCODE -ne 0) {
                                throw "Docker tag gagal"
                            }

                            docker push $latest
                            if ($LASTEXITCODE -ne 0) {
                                throw "Push latest gagal"
                            }
                        }
                        finally {
                            docker logout
                        }
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

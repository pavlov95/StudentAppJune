pipeline {
    agent any

    tools {
        nodejs 'NodeJS 20'
    }

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Start application') {
            steps {
                powershell '''
                    $process = Start-Process npm -ArgumentList "start" -PassThru
                    $process.Id | Out-File app.pid
                '''
            }
        }

        stage('Wait for application') {
            steps {
                powershell '''
                    $maxAttempts = 20

                    for ($i = 1; $i -le $maxAttempts; $i++) {
                        try {
                            Invoke-WebRequest -Uri "http://localhost:8888/" -UseBasicParsing | Out-Null
                            Write-Host "Application is ready."
                            exit 0
                        }
                        catch {
                            Write-Host "Waiting for application..."
                            Start-Sleep -Seconds 1
                        }
                    }

                    throw "Application did not start."
                '''
            }
        }

        stage('Run tests') {
            steps {
                bat 'npm test'
            }
        }
    }

    post {
        always {
            powershell '''
                if (Test-Path app.pid) {
                    $pid = Get-Content app.pid

                    Stop-Process -Id $pid -Force -ErrorAction SilentlyContinue
                    Remove-Item app.pid
                }
            '''
        }
    }
}
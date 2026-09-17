pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Check Node.js') {
            steps {
                bat 'node --version'
                bat 'npm --version'
            }
        }

        stage('Install dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Start application') {
            steps {
                withEnv(['JENKINS_NODE_COOKIE=dontKillMe']) {
                    powershell '''
                        $process = Start-Process `
                            -FilePath "npm.cmd" `
                            -ArgumentList "start" `
                            -WorkingDirectory $env:WORKSPACE `
                            -RedirectStandardOutput "$env:WORKSPACE\\app-out.log" `
                            -RedirectStandardError "$env:WORKSPACE\\app-error.log" `
                            -PassThru

                        $process.Id | Set-Content "$env:WORKSPACE\\app.pid"

                        Write-Host "Application started with PID $($process.Id)"
                    '''
                }
            }
        }

        stage('Wait for application') {
            steps {
                powershell '''
                    for ($i = 1; $i -le 20; $i++) {
                        try {
                            $response = Invoke-WebRequest `
                                -Uri "http://localhost:8888/" `
                                -UseBasicParsing

                            Write-Host "Application is ready."
                            exit 0
                        }
                        catch {
                            Write-Host "Waiting for application..."
                            Start-Sleep -Seconds 1
                        }
                    }

                    Write-Host "Application failed to start."

                    if (Test-Path "$env:WORKSPACE\\app-out.log") {
                        Write-Host "--- stdout ---"
                        Get-Content "$env:WORKSPACE\\app-out.log"
                    }

                    if (Test-Path "$env:WORKSPACE\\app-error.log") {
                        Write-Host "--- stderr ---"
                        Get-Content "$env:WORKSPACE\\app-error.log"
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
                if (Test-Path "$env:WORKSPACE\\app.pid") {
                    $processId = Get-Content "$env:WORKSPACE\\app.pid"

                    Write-Host "Stopping application PID $processId"

                    taskkill /PID $processId /T /F 2>$null

                    Remove-Item "$env:WORKSPACE\\app.pid" -Force
                }
            '''
        }
    }
}
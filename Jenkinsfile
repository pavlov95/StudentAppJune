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
                bat 'start /B npm start'
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
}
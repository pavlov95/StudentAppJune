pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Test') {
            steps {
                powershell '''
                    $app = Start-Process node `
                        -ArgumentList "index.js" `
                        -PassThru

                    Start-Sleep -Seconds 3

                    try {
                        npm.cmd test

                        if ($LASTEXITCODE -ne 0) {
                            exit $LASTEXITCODE
                        }
                    }
                    finally {
                        Stop-Process -Id $app.Id -Force -ErrorAction SilentlyContinue
                    }
                '''
            }
        }
    }
}
pipeline {
    agent any

    stages {
        stage('Install dependencies') {
            steps {
                bat 'npm ci'
            }
        }

        stage('Run tests') {
            steps {
                powershell '''
                    $app = Start-Process node `
                        -ArgumentList "index.js", "8888" `
                        -PassThru

                    Start-Sleep -Seconds 2

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
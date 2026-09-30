pipeline {
    agent any

    options {
        timestamps()
    }

    parameters {
        string(
            name: 'HTTP_SERVER_ROOT',
            defaultValue: 'C:\\inetpub\\wwwroot',
            description: 'Local web server document root on the Windows Jenkins agent'
        )
        string(
            name: 'HTTP_BASE_URL',
            defaultValue: 'http://localhost',
            description: 'Base URL served by the local HTTP server'
        )
    }

    stages {
        stage('Initialize') {
            steps {
                script {
                    currentBuild.displayName = "automated http validation #${env.BUILD_NUMBER}"
                    currentBuild.description = 'Deploy static files and validate the local HTTP endpoint'
                }
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy static files') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    $source = $env:WORKSPACE
                    $destination = $env:HTTP_SERVER_ROOT

                    if (-not (Test-Path (Join-Path $source 'index.html'))) {
                        throw "index.html was not found in the workspace: $source"
                    }

                    New-Item -ItemType Directory -Path $destination -Force | Out-Null
                    & robocopy $source $destination /E /XD (Join-Path $source '.git') /XF 'Jenkinsfile'
                    $copyExitCode = $LASTEXITCODE
                    if ($copyExitCode -ge 8) {
                        throw "Robocopy failed with exit code $copyExitCode"
                    }

                    Write-Host "Static files deployed to $destination"
                '''
            }
        }

        stage('Validate HTTP') {
            steps {
                powershell '''
                    $ErrorActionPreference = 'Stop'
                    $url = $env:HTTP_BASE_URL.TrimEnd('/') + '/index.html'
                    $response = Invoke-WebRequest -Uri $url -Method Get -UseBasicParsing -TimeoutSec 20

                    if ([int]$response.StatusCode -ne 200) {
                        throw "Expected HTTP 200 from $url, received $($response.StatusCode)"
                    }
                    if ($response.Content -notmatch '(?i)<html') {
                        throw "The response from $url does not look like an HTML page"
                    }

                    Write-Host "HTTP validation passed: $url returned $($response.StatusCode)"
                '''
            }
        }
    }
}

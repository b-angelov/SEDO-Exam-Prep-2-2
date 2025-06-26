pipeline{
    agent any

    stages{
        stage("Conditional wrapper"){
            when{
                expression{
                    return env.BRANCH_NAME == 'main' || env.BRANCH_NAME.startsWith('feature')
                }
            }

            stages{
                stage("Checkout SCM"){
                    steps{
                    checkout scm
                }
                }

                    stage('Check & Install .NET 6') {
                        steps {
                            powershell '''
                            $dotnet = Get-Command dotnet -ErrorAction SilentlyContinue

                            if (-not $dotnet) {
                                Write-Output "❌ .NET CLI not found. Installing..."
                            } else {
                                $sdks = & dotnet --list-sdks
                                if ($sdks -match "^6\\.0\\.") {
                                Write-Output "✅ .NET 6.0 SDK already installed."
                                return
                                } else {
                                Write-Output "❌ .NET 6.0 SDK not found. Installing..."
                                }
                            }

                            # Download and install .NET 6.0
                            $url = "https://dot.net/v1/dotnet-install.ps1"
                            $script = "dotnet-install.ps1"
                            Invoke-WebRequest $url -OutFile $script
                            .\\$script -Version 6.0.416 -InstallDir "C:\\dotnet"

                            # Add to PATH for current session
                            $env:PATH = "C:\\dotnet;" + $env:PATH

                            # Confirm
                            & C:\\dotnet\\dotnet --version
                            '''
                        }
                        }

                    stage('Build') {
                        steps{
                            bat 'dotnet build'
                        }
                    }

                    stage('Test') {
                        steps {
                            bat 'dotnet test --no-build --verbosity normal'
                        }
                    }
            }

        }
    }
}
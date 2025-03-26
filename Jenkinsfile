pipeline {
    agent any
    environment {
        AWS_REGION = "ap-northeast-1"
        ZIP_FILE = 'dotnet-mvc-app.zip'
    }
    stages {
        stage('Checkout Code') {
            // Check out the main branch from the repository
            steps {
                cleanWs()  // Cleans up the workspace before cloning
                checkout scm

                // Show last commit info
                sh """
                    echo 'Last Commit Info:'
                    git log -1 --pretty=format:'%h - %an: %s (%cr)'
                """
                // show files
                sh "ls -la"
            }
        }
        
        stage('Trigger AWS CodeArtifact Authentication') {
            steps {
                script {
                    def authBuild = build job: 'Common/aws-codeartifact-auth-pipeline', wait: true
                    def token = authBuild.description.tokenize('=')[1]  // Extract token

                    echo "Using AWS CodeArtifact token securely.."

                    // Store token in memory for later use
                    env.CODEARTIFACT_AUTH_TOKEN = token

                    echo "Check NUGET list source.."

                    sh """
                        dotnet nuget list source
                    """
                }
            }
        }
        
        stage('Build & Publish Application') {
            // Build and publish the .NET application using dotnet CLI
            steps {
                script {
                    withAWS(region: "${AWS_REGION}") {  // Ensures all AWS CLI commands use correct credentials
                        sh """
                            cd WebApp
                            dotnet publish -c Release -o ../publish -v d
                            ls -la
                        """
                    }
                }
            }
        }

        stage('Package Application') {
            // Package the application into a zip file for deployment
            steps {
                script {
                    withAWS(region: "${AWS_REGION}") {  // Ensures all AWS CLI commands use correct credentials
                        sh """
                            ls -la
                            cd publish
                            zip -r ../${ZIP_FILE} .
                            cd ..  # Navigate back to the parent directory
                            ls -la
                        """
                    }
                }
            }
        }
    }
}

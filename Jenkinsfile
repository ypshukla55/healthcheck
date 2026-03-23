pipeline {
    agent any

    // Triggers the build when a PR is merged into the 'main' branch
    triggers {
        githubPullRequests(
            spec: 'H/5 * * * *',
            triggerMode: 'HEAVY_DUTY_SYNCING',
            events: [ [ $class: 'GitHubPRMergedEvent' ] ]
        )
    }

    environment {
        VAULT_URL = 'http://localhost:8200'
        // The path from your screenshot
        VAULT_PATH = 'secret/data/jenkins/dev'
    }

    stages {
        stage('Fetch Secrets from Vault') {
            steps {
                // withVault bridges Jenkins with your local Vault instance
                withVault(
                    configuration: [
                        vaultUrl: "${env.VAULT_URL}",
                        vaultCredentialId: 'vault-token-id', // ID of the credential you created in Jenkins
                        engineVersion: 2
                    ],
                    vaultSecrets: [[
                        path: "${env.VAULT_PATH}",
                        secretValues: [
                            [envVar: 'MY_USER', vaultKey: 'username'],
                            [envVar: 'MY_PASS', vaultKey: 'password']
                        ]
                    ]]
                ) {
                    script {
                        echo "Successfully retrieved username: ${env.MY_USER}"
                        // Password is automatically masked by Jenkins (****)
                    }
                }
            }
        }

        stage('Pull Latest Code') {
            steps {
                // Pulls the latest code from the branch that was just merged
                git branch: 'main', 
                    url: 'https://github.com/ypshukla55/healthcheck'
                
                echo "Latest code pulled successfully."
            }
        }

        stage('Deploy/Build') {
            steps {
                sh "echo 'Running build using user: ${env.MY_USER}'"
                // Add your build or deployment commands here
            }
        }
    }

    post {
        always {
            cleanWs() // Cleanup workspace to remove any sensitive traces
        }
    }
}

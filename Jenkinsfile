pipeline {
    agent any

    parameters {
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to deploy')
        choice(name: 'ENVIRONMENT', choices: ['DEV','PROD'], description: 'Select deployment environment')
    }

    environment {
        // Git
        GIT_REPO = 'https://github.com/ypshukla55/healthcheck.git'
        // GIT_CREDENTIALS_ID = 'git-ssh-creds'

        // Default values (overwritten later)
        VAULT_URL = 'http://127.0.0.1:8200'
        VAULT_ROLE_ID = '34d225f8-6067-c74c-f3d9-b4bac3619e44'
        VAULT_SECRET_PATH = 'secret/jenkins/dev'
        TARGET_SERVER = '192.168.1.105'
        DEPLOY_DIR = '/apps/project'
    }

    stages {

        stage('Set Environment Config') {
            steps {
                script {

                    if (params.ENVIRONMENT == "DEV") {

                        env.VAULT_URL = "http://127.0.0.1:8200"
                        env.VAULT_ROLE_ID = "34d225f8-6067-c74c-f3d9-b4bac3619e44"
                        env.VAULT_SECRET_PATH = "secret/jenkins/dev"
                        env.TARGET_SERVER = "192.168.1.105"

                    } else {

                        env.VAULT_URL = "https://vault.company.com:8200"
                        env.VAULT_ROLE_ID = "PROD_ROLE_ID"
                        env.VAULT_SECRET_PATH = "kv/project/prod/app"
                        env.TARGET_SERVER = "prod-server.company.com"

                    }

                    echo "Deploying to ${params.ENVIRONMENT}"
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: "${params.BRANCH}",
                    // credentialsId: "${GIT_CREDENTIALS_ID}",
                    url: "${GIT_REPO}"
            }
        }

        stage('Fetch Credentials from Vault') {
            steps {
                withVault(
                    configuration: [
                        vaultUrl: "${env.VAULT_URL}",
                        // Wrap these in the configuration block
                        roleId: "${env.VAULT_ROLE_ID}",
                        secretId: credentials('94772171-18b8-acbd-666e-ba4ad7adf502'),
                        engineVersion: 2
                    ],
                    vaultSecrets: [[
                        // For KV-V2, Vault expects 'data' in the path: secret/data/...
                        path: "secret/data/jenkins/dev", 
                        secretValues: [
                            [envVar: 'DEPLOY_USER', vaultKey: 'username'],
                            [envVar: 'DEPLOY_PASSWORD', vaultKey: 'password']
                        ]
                    ]]
                ) {
                    script {
                        if (!env.DEPLOY_USER || !env.DEPLOY_PASSWORD) {
                            error("Vault credentials not retrieved. Check if path 'secret/jenkins/dev' exists.")
                        }
                        echo "Vault credentials retrieved successfully."
                    }
                }
            }
        }

        stage('Deploy Application') {

            steps {

                sh """

                echo "Deploying branch ${params.BRANCH} to ${TARGET_SERVER}"

                sshpass -p '${DEPLOY_PASSWORD}' ssh -o StrictHostKeyChecking=no ${DEPLOY_USER}@${TARGET_SERVER} "mkdir -p ${DEPLOY_DIR}"

                sshpass -p '${DEPLOY_PASSWORD}' scp -r * ${DEPLOY_USER}@${TARGET_SERVER}:${DEPLOY_DIR}/

                sshpass -p '${DEPLOY_PASSWORD}' ssh ${DEPLOY_USER}@${TARGET_SERVER} "chmod -R 775 ${DEPLOY_DIR}"

                """

            }
        }
    }

    post {

        always {
            cleanWs()
        }

        success {
            echo "Deployment Successful"
        }

        failure {
            echo "Deployment Failed"
        }
    }
}

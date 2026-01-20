pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    parameters {
        choice(
            name: 'TARGET_ENV',
            choices: ['dev', 'uat', 'prod'],
            description: 'Target environment'
        )
    }

    triggers {
        pollSCM('H/5 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Decrypt SSH Key') {
            steps {
                sh '''
                  cd ansible

                  ansible-vault decrypt files/deploy_key.vault \
                    --vault-password-file secrets/vault-pass

                  chmod 600 files/deploy_key
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                  cd ansible

                  ansible-playbook playbooks/deploy.yml \
                    -i inventories/${TARGET_ENV}.yml \
                    --private-key files/deploy_key
                '''
            }
        }
    }

    post {
        always {
            sh '''
              rm -f ansible/files/deploy_key
            '''
            cleanWs()
        }
        success {
            echo "Deployment to ${params.TARGET_ENV} completed successfully"
        }
        failure {
            echo "Deployment to ${params.TARGET_ENV} failed"
        }
    }
}

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    triggers {
        pollSCM('H/2 * * * *')
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
                      --vault-password-file secrets/vault-pass \
                      --output files/deploy_key
                    chmod 600 files/deploy_key
                '''
            }
        }

        stage('Deploy') {
            when {
                branch 'main'
            }
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
            sh 'rm -f ansible/files/deploy_key || true'
            cleanWs()
        }
        success {
            echo 'Deployment completed successfully'
        }
        failure {
            echo 'Deployment failed'
        }
    }
}
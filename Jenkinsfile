pipeline {
    agent any

    environment {
        VAULT_PASS_FILE = "${WORKSPACE}/.vault_pass"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Vault Password') {
            steps {
                withCredentials([
                    string(credentialsId: 'ANSIBLE_VAULT_PASSWORD', variable: 'VAULT_PASS')
                ]) {
                    sh '''
                        echo "$VAULT_PASS" > .vault_pass
                        chmod 600 .vault_pass
                    '''
                }
            }
        }

        stage('Deploy to Rocky VM') {
            steps {
                sshagent(credentials: ['rocky_ssh_key']) {
                    sh '''
                        cd ansible
                        ansible-playbook playbooks/deploy.yml \
                          -i inventories/dev.ini \
                          --vault-password-file ../.vault_pass
                    '''
                }
            }
        }
    }

    post {
        always {
            sh 'rm -f .vault_pass'
        }
    }
}


pipeline {
    agent any

    triggers {
        githubPush()
    }

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

        stage('Deploy to DEV') {
            steps {
                sshagent(credentials: ['rocky_ssh_key']) {
                    sh '''
                      cd ansible
                      ansible-playbook playbooks/deploy.yml \
                        -i inventories/dev.yml \
                        --vault-password-file ../.vault_pass
                    '''
                }
            }
        }

        // Optional gates / promotions
        stage('Deploy to UAT') {
            when { branch 'main' }
            steps {
                sshagent(credentials: ['rocky_ssh_key']) {
                    sh '''
                      cd ansible
                      ansible-playbook playbooks/deploy.yml \
                        -i inventories/uat.yml \
                        --vault-password-file ../.vault_pass
                    '''
                }
            }
        }

        stage('Deploy to PROD') {
            when { branch 'main' }
            steps {
                input message: 'Approve PROD deployment?'
                sshagent(credentials: ['rocky_ssh_key']) {
                    sh '''
                      cd ansible
                      ansible-playbook playbooks/deploy.yml \
                        -i inventories/prod.yml \
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


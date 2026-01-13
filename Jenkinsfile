pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        VAULT_PASS_FILE = "${WORKSPACE}/.vault_pass"
	SSH_AUTH_SOCK = ''
    }

    stages {

        stage('Checkout') {
            steps {
                // RUNS AS: jenkins (Ubuntu)
                checkout scm
            }
        }

        stage('Prepare Vault Password') {
            steps {
                // RUNS AS: jenkins (Ubuntu)
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
                // RUNS AS: jenkins (Ubuntu)
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

    post {
        always {
            // RUNS AS: jenkins (Ubuntu)
            sh 'rm -f .vault_pass'
        }
    }
}


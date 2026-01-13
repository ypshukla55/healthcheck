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

        stage('Deploy to Dev') {
            steps {
                sshagent(credentials: ['rocky_ssh_key']) {
                    sh '''
                        cd ansible
                        ansible-playbook playbooks/deploy.yml \
                          -i inventories/dev.yml
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Deployment completed successfully'
        }
        failure {
            echo 'Deployment failed'
        }
        always {
            cleanWs()
        }
    }
}


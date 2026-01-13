pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        timestamps()
    }

    triggers {
        // Jenkins polls GitHub every 2 minutes for changes
        pollSCM('H/2 * * * *')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Deploy to Dev') {
            when {
                branch 'main'
            }
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


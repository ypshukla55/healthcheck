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
                sh '''
                cd ansible

                # prepare key BEFORE ansible-playbook starts
                cp files/test_ansible_key /tmp/ansible_test_key
                chmod 600 /tmp/ansible_test_key

                ansible-playbook playbooks/deploy.yml \
                    -i inventories/dev.yml \
                    --private-key /tmp/ansible_test_key

                rm -f /tmp/ansible_test_key
                '''
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


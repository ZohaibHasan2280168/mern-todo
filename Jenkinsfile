pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub repository...'
                git branch: 'main', url: 'https://github.com/ZohaibHasan2280168/mern-todo.git'
            }
        }

        stage('Deploy Microservices to ALL VMs via Ansible') {
            steps {
                echo 'Deploying application across all VMs in inventory...'
                sh '''
                    # Single Ansible execution that targets ALL VMs inside inventory.ini
                    ansible-playbook -i inventory.ini deploy-app.yml
                '''
            }
        }
    }

    post {
        success {
            echo 'Microservices successfully deployed to ALL target VMs!'
        }
        failure {
            echo 'Deployment Failed on one or more VMs!'
        }
    }
}

pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub...'
                git branch: 'main', url: 'https://github.com/ZohaibHasan2280168/mern-todo.git'
            }
        }

        stage('Deploy to All VMs via Ansible') {
            steps {
                echo 'Deploying MERN stack across all target VMs...'
                sh 'ansible-playbook -i inventory.ini deploy-app.yml'
            }
        }
    }

    post {
        success {
            echo 'App successfully deployed on all VMs!'
        }
        failure {
            echo 'Deployment failed on one or more VMs!'
        }
    }
}

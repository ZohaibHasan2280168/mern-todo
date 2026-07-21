pipeline {
    agent any

    environment {
        TARGET_IP = '20.205.14.246'
        TARGET_USER = 'azureuser'
        SSH_CMD = "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${TARGET_USER}@${TARGET_IP}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest production assets from Git Repository...'
                git branch: 'main', url: 'https://github.com/ZohaibHasan2280168/mern-todo.git'
            }
        }

        stage('Deploy Components to Remote VM') {
            steps {
                echo "Deploying microservices to Remote VM (${env.TARGET_IP})..."
                sh """
                    ${env.SSH_CMD} '
                        # 1. Project Directory Check / Clone
                        if [ ! -d "mern-todo" ]; then
                            git clone https://github.com/ZohaibHasan2280168/mern-todo.git
                        fi
                        
                        cd mern-todo
                        git pull origin main

                        # 2. Network setup
                        docker network create mern_network || true

                        # 3. Database Tier (MongoDB)
                        docker rm -f mongodb_container || true
                        docker run -d --name mongodb_container --network mern_network -p 27017:27017 -v mongo_data:/data/db mongo:6.0

                        # 4. Wait for MongoDB
                        echo "Waiting for MongoDB..."
                        sleep 5

                        # 5. Backend Tier
                        docker rm -f backend_container || true
                        docker build -t mern-backend -f server/Dockerfile .
                        docker run -d --name backend_container --network mern_network -p 5005:5000 -e MONGODB_ATLAS_CONNECTION=mongodb://mongodb_container:27017/todo -e PORT=5000 mern-backend

                        # 6. Injecting Nginx Config
                        cat << "EOF" > client/default.conf
server {
    listen 80;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files \$uri \$uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend_container:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade \$http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host \$host;
        proxy_set_header X-Real-IP \$remote_addr;
        proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto \$scheme;
    }
}
EOF

                        # 7. Frontend Tier Deployment
                        docker stop frontend_container || true
                        docker rm -f frontend_container || true
                        docker build -t mern-frontend -f client/Dockerfile .
                        docker run -d --name frontend_container --network mern_network -p 8082:80 mern-frontend
                    '
                """
            }
        }

        stage('Post-Deployment Verification') {
            steps {
                echo 'Validating remote container logs...'
                sh "${env.SSH_CMD} 'docker ps'"
            }
        }
    }

    post {
        success {
            echo "Continuous Deployment finalized successfully! App is live at http://${env.TARGET_IP}:8082"
        }
        failure {
            echo 'Deployment Execution Failure state detected. Check logs.'
        }
    }
}
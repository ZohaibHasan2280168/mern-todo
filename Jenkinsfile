pipeline {
    agent any

    environment {
        REMOTE_VM_IP = '20.205.14.246'
        REMOTE_USER  = 'azureuser'
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
                echo "Deploying microservices to Remote VM (${REMOTE_VM_IP})..."
                sh '''
                    # 1. Setup Remote SSH Public Key dynamically on target host
                    ssh -o StrictHostKeyChecking=no ${REMOTE_USER}@${REMOTE_VM_IP} "mkdir -p ~/.ssh && echo 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC4fUIs+YXng+IIYwaO5irNweJGFURCyYPbD/ieOPB9gfc02weiG/kagK8Tqs96OtqdsCEkDVBYB6fSzJ1iLLz3U8BG9WOC99OFhvFeioAY+/F/03j+Dp9tlE7PWBZwK3SV2KrLHByQChhR0uDMTXclqJ9tET4DCblh+oypoXIPZ4tlKfr5HTLkzTALPAdb8xJNjhJ5DxNbh23Ouq1Bx1EU6rxhR5Glxw+RX/kdQGEDfDWO4nemBaFuI6J4uzFvQfo7S0aoha4N3milEe8dLc977tNZlfU1WDwxyMPlS2MyTlZf5rlZdg3F3p6FanHv9BXofIJAnDcNv9CG3dbLxDLPMvDajclahKbE3I6m3g76SywLGhB9/Wj3xVmz+WmOX5491+pNx3JuaW96QSXqUBPgbWEqfJ+3ADErKnVjqcMnnndm8WTw6v2gjwRI/qAs1mxiCHFs2k9rfeKwRXgXxZxy7W36eJM/yIf4NMlEzrDwMIXmoqQHqnp8WH2J9M8Aj80=' >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys" || true

                    # 2. Execute Microservices Deployment
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_VM_IP} << 'EOF'
                        if [ ! -d "mern-todo" ]; then
                            git clone https://github.com/ZohaibHasan2280168/mern-todo.git
                        fi
                        
                        cd mern-todo
                        git pull origin main

                        # Setup Docker Network
                        docker network create mern_network || true

                        # Database Tier (MongoDB)
                        docker rm -f mongodb_container || true
                        docker run -d --name mongodb_container --network mern_network -p 27017:27017 -v mongo_data:/data/db mongo:6.0

                        echo "Waiting for MongoDB..."
                        sleep 5

                        # Backend Tier
                        docker rm -f backend_container || true
                        docker build -t mern-backend -f server/Dockerfile .
                        docker run -d --name backend_container --network mern_network -p 5005:5000 -e MONGODB_ATLAS_CONNECTION=mongodb://mongodb_container:27017/todo -e PORT=5000 mern-backend

                        # Frontend Nginx Config
                        cat << "NGINX_CONF" > client/default.conf
server {
    listen 80;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend_container:5000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
NGINX_CONF

                        # Frontend Tier Deployment
                        docker stop frontend_container || true
                        docker rm -f frontend_container || true
                        docker build -t mern-frontend -f client/Dockerfile .
                        docker run -d --name frontend_container --network mern_network -p 8082:80 mern-frontend
EOF
                '''
            }
        }
    }

    post {
        failure {
            echo 'Deployment Execution Failure state detected. Check logs.'
        }
        success {
            echo 'Microservices successfully deployed to target VM!'
        }
    }
}
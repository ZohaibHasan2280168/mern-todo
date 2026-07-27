pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest code from GitHub repository...'
                git branch: 'main', url: 'https://github.com/ZohaibHasan2280168/mern-todo.git'
            }
        }

        stage('Deploy Microservices') {
            steps {
                echo 'Deploying Microservices using Host System Docker...'
                sh '''
                    # Install Docker CLI inside Jenkins container if missing
                    if ! command -v docker &> /dev/null; then
                        echo "Docker CLI not found inside Jenkins. Installing..."
                        apt-get update && apt-get install -y docker.io || true
                    fi

                    # Execute Docker commands
                    docker network create mern_network || true

                    # 1. Database Tier (MongoDB)
                    docker rm -f mongodb_container || true
                    docker run -d --name mongodb_container --network mern_network -p 27017:27017 -v mongo_data:/data/db mongo:6.0

                    echo "Waiting for Database setup..."
                    sleep 5

                    # 2. Backend Tier
                    docker rm -f backend_container || true
                    docker build -t mern-backend -f server/Dockerfile .
                    docker run -d --name backend_container --network mern_network -p 5005:5000 -e MONGODB_ATLAS_CONNECTION=mongodb://mongodb_container:27017/todo -e PORT=5000 mern-backend

                    # 3. Frontend Nginx Setup
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

                    # 4. Frontend Tier
                    docker rm -f frontend_container || true
                    docker build -t mern-frontend -f client/Dockerfile .
                    docker run -d --name frontend_container --network mern_network -p 8082:80 mern-frontend
                '''
            }
        }
    }

    post {
        success {
            echo 'Microservices successfully deployed!'
        }
        failure {
            echo 'Deployment Failed!'
        }
    }
}

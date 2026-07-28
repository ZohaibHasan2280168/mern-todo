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
                    # Create Docker internal network if not exists
                    docker network create mern_network || true

                    # 1. Database Tier (MongoDB)
                    echo "--- Deploying MongoDB ---"
                    docker rm -f mongodb_container || true
                    docker run -d --name mongodb_container \
                      --network mern_network \
                      -p 27017:27017 \
                      -v mongo_data:/data/db \
                      mongo:6.0

                    echo "Waiting for Database to start..."
                    sleep 5

                    # 2. Backend Tier
                    echo "--- Building and Deploying Backend ---"
                    docker rm -f backend_container || true
                    docker build -t mern-backend -f server/Dockerfile .
                    docker run -d --name backend_container \
                      --network mern_network \
                      -p 5005:5000 \
                      -e MONGODB_ATLAS_CONNECTION=mongodb://mongodb_container:27017/todo \
                      -e PORT=5000 \
                      mern-backend

                    # 3. Create Nginx Configuration for Frontend
                    echo "--- Preparing Nginx Config ---"
                    cat << "NGINX_CONF" > client/default.conf
server {
    listen 80;
    
    location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://backend_container:5000/;
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
                    echo "--- Building and Deploying Frontend ---"
                    docker rm -f frontend_container || true
                    docker build -t mern-frontend -f client/Dockerfile .
                    docker run -d --name frontend_container \
                      --network mern_network \
                      -p 8082:80 \
                      mern-frontend
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

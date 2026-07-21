pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                echo 'Pulling latest production assets from secure Git Repository...'
                git branch: 'main', url: 'https://github.com/ZohaibHasan2280168/mern-todo.git'
            }
        }

        stage('Build & Deploy Components') {
            steps {
                echo 'Building and starting microservices from root context...'
                sh '''
                    # 1. Network setup
                    docker network create mern_network || true

                    # 2. Database Tier (MongoDB)
                    docker rm -f mongodb_container || true
                    docker run -d --name mongodb_container --network mern_network -p 27017:27017 -v mongo_data:/data/db mongo:6.0

                    # 3. Wait for MongoDB to fully initialize
                    echo "Waiting for MongoDB to fully initialize..."
                    sleep 5

                    # 4. Backend/Server Tier
                    docker rm -f backend_container || true
                    docker build -t mern-backend -f server/Dockerfile .
                    docker run -d --name backend_container --network mern_network -p 5005:5000 -e MONGODB_ATLAS_CONNECTION=mongodb://mongodb_container:27017/todo -e PORT=5000 mern-backend

                    # 5. Injecting Reverse Proxy into Client Workspace
                    cat << 'EOF' > client/default.conf
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
        proxy_set_header Upgrade $upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
EOF

                    # 6. Frontend/Client Tier Deployment
                    docker stop frontend_container || true
                    docker rm -f frontend_container || true
                    
                    docker build -t mern-frontend -f client/Dockerfile .
                    docker run -d --name frontend_container --network mern_network -p 8082:80 mern-frontend
                '''
            }
        }
        
        stage('Post-Deployment Verification') {
            steps {
                echo 'Validating operational status logs...'
                sh 'docker ps'
            }
        }
    }
    
    post {
        success {
            echo 'Continuous Deployment Sequence finalized successfully. Deployment is live.'
        }
        failure {
            echo 'Deployment Execution Failure state detected. Inspect active agent console logs.'
        }
    }
}
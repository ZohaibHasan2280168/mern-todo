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
                    # 1. Private Key Temp File Write Karo
                    cat << 'EOF' > /tmp/target_vm_key.pem
-----BEGIN RSA PRIVATE KEY-----
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
-----END RSA PRIVATE KEY-----
EOF

                    # 2. Strict Permissions Set Karo
                    chmod 400 /tmp/target_vm_key.pem

                    # 3. Direct Key Based SSH Execution
                    ssh -i /tmp/target_vm_key.pem -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_VM_IP} << 'EOF'
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
                    # 4. Cleanup Key After Job Execution
                    rm -f /tmp/target_vm_key.pem
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
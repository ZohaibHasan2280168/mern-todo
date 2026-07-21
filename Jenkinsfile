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
-----BEGIN OPENSSH PRIVATE KEY-----
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
V8ncUMttBsxFAAAACWthbGlAa2FsaQE=
-----END OPENSSH PRIVATE KEY-----
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
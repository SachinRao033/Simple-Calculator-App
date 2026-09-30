pipeline {
    agent any

    environment {
        PROJECT_DIR = "/home/ubuntu/Simple-Calculator-App"
        VITE_API_URL = "http://13.205.69.179:8000"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Copy Project') {
            steps {
                sh '''
                    echo "======================================"
                    echo "Jenkins Workspace: ${WORKSPACE}"
                    echo "Deployment Directory: ${PROJECT_DIR}"
                    echo "======================================"

                    sudo mkdir -p "${PROJECT_DIR}"

                    # Copy project files without deleting existing files
                    sudo cp -r "${WORKSPACE}/." "${PROJECT_DIR}/"

                    sudo chown -R jenkins:jenkins "${PROJECT_DIR}"

                    echo "Project copied successfully"

                    echo "===== Project Files ====="
                    ls -la "${PROJECT_DIR}"
                '''
            }
        }

        stage('Create Environment Files') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    cat > .env <<EOF
POSTGRES_DB=calculator_db
POSTGRES_USER=calculator
POSTGRES_PASSWORD=calculator123
DATABASE_URL=postgresql://calculator:calculator123@postgres:5432/calculator_db
CORS_ORIGINS=http://13.205.69.179:3000
VITE_API_URL=http://13.205.69.179:8000
EOF

                    echo "Environment file created"

                    echo "===== .env ====="
                    cat .env
                '''
            }
        }

        stage('Stop Old Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "===== Stopping Old Containers ====="

                    docker compose down || true

                    echo "Old containers stopped"
                '''
            }
        }

        stage('Build Docker Images') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "======================================"
                    echo "Building Docker Images"
                    echo "======================================"

                    docker compose build --no-cache
                '''
            }
        }

        stage('Deploy Containers') {
            steps {
                sh '''
                    cd "${PROJECT_DIR}"

                    echo "======================================"
                    echo "Starting Containers"
                    echo "======================================"

                    docker compose up -d

                    echo "Containers started"
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for services..."
                    sleep 20

                    cd "${PROJECT_DIR}"

                    echo "======================================"
                    echo "Docker Compose Status"
                    echo "======================================"

                    docker compose ps

                    echo "======================================"
                    echo "Running Containers"
                    echo "======================================"

                    docker ps

                    echo "======================================"
                    echo "Backend Health Check"
                    echo "======================================"

                    curl -f http://localhost:8000/docs > /dev/null

                    echo "Backend is healthy"

                    echo "======================================"
                    echo "Frontend Health Check"
                    echo "======================================"

                    curl -f http://localhost:3000 > /dev/null

                    echo "Frontend is healthy"

                    echo "======================================"
                    echo "Calculator API Test"
                    echo "======================================"

                    curl -f -X POST \
                        http://localhost:8000/api/calculate \
                        -H "Content-Type: application/json" \
                        -d '{"expression":"589*6"}'

                    echo ""
                    echo "Calculator API is working"

                    echo "======================================"
                    echo "Application deployed successfully!"
                    echo "======================================"
                '''
            }
        }
    }

    post {

        success {
            echo "SUCCESS: Calculator Web App deployed successfully!"
            echo "Frontend: http://13.205.69.179:3000"
            echo "Backend:  http://13.205.69.179:8000/docs"
        }

        failure {
            echo "FAILED: Deployment failed. Check Jenkins console output."
        }

        always {
            sh '''
                sudo chown -R jenkins:jenkins "${PROJECT_DIR}" || true
                docker image prune -f || true
            '''
        }
    }

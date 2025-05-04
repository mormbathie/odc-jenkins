pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIALS = 'docker_hub'
        DOCKERHUB_USER = 'mormbathie'
        NODE_PATH = '/var/lib/jenkins/.nvm/versions/node/v18.20.0/bin' // 🧠 À ajuster selon ta version exacte
    }

    stages {
        stage('Checkout') {
            steps {
                echo "📥 Clonage du dépôt Git"
                checkout scm
            }
        }

        stage('Build & Test Backend (Django)') {
            steps {
                dir('Backend/odc') {
                    echo "⚙️ Création de l'environnement virtuel et test de Django"
                    sh '''
                        python3 -m venv venv
                        . venv/bin/activate
                        pip install --upgrade pip
                        pip install -r requirements.txt
                        python manage.py test
                    '''
                }
            }
        }

        stage('Build & Test Frontend (React)') {
            steps {
                dir('Frontend') {
                    echo "⚙️ Installation et test du frontend React"
                    sh '''
                    export NVM_DIR="/var/lib/jenkins/.nvm"
                    [ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
                    nvm use 18
                
                    node -v
                    npm install
                    npm audit fix || true
                    npm run build
                    '''

                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🐳 Construction des images Docker"
                sh '''
                    docker build -t ${DOCKERHUB_USER}/odc_backend:latest -f ./Backend/odc/Dockerfile ./Backend/odc
                    docker build -t ${DOCKERHUB_USER}/odc_frontend:latest ./Frontend
                '''
            }
        }

        stage('Push Docker Images') {
            steps {
                echo "🚀 Envoi des images Docker sur Docker Hub"
                withCredentials([usernamePassword(credentialsId: "${DOCKER_HUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_USER/odc_backend:latest
                        docker push $DOCKER_USER/odc_frontend:latest
                    '''
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                echo "🚀 Lancement via Docker Compose"
                sh '''
                    docker-compose down || true
                    docker-compose build
                    docker-compose up -d
                '''
            }
        }
    }

    post {
        success {
            echo "✅ CI/CD terminé avec succès"
        }
        failure {
            echo "❌ Échec du pipeline"
        }
    }
}

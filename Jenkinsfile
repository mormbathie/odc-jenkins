pipeline {
    agent any

    environment {
        // Configuration de SonarQube (Nom donné dans Jenkins > Manage Jenkins > Tools > SonarQube)
        SONARQUBE_ENV = 'DefaultScanner'
        // Identifiants DockerHub enregistrés dans Jenkins (Manage Jenkins > Credentials)
        DOCKER_HUB_CREDENTIALS_ID = 'dockerhub-creds'
        // Nom du repo DockerHub
        DOCKER_HUB_REPO = 'mormbathie/odc'
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Clonage du dépôt Git'
                checkout scm
            }
        }

        stage('Analyse SonarQube') {
            steps {
                echo '🔍 Analyse du code avec SonarQube'
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    dir('Backend/odc') {
                        sh 'sonar-scanner -Dsonar.projectKey=fileRouge -Dsonar.sources=. -Dsonar.host.url=http://localhost:9000'
                    }
                }
            }
        }

        stage('Build & Test Backend (Django)') {
            steps {
                dir('Backend/odc') {
                    echo '🛠️ Backend - Installation des dépendances'
                    sh 'python3 -m venv venv && . venv/bin/activate && pip install -r requirements.txt'
                    echo '🧪 Backend - Exécution des tests'
                    sh '. venv/bin/activate && python manage.py test'
                }
            }
        }

        stage('Build & Test Frontend (React)') {
            steps {
                dir('Frontend') {
                    echo '🛠️ Frontend - Installation des dépendances'
                    sh 'npm install'
                    echo '🧪 Frontend - Exécution des tests'
                    sh 'npm test'
                }
            }
        }

        stage('Build Docker Images') {
            steps {
                echo '🐳 Construction des images Docker'
                sh 'docker build -t $DOCKER_HUB_REPO-backend:latest ./Backend/odc'
                sh 'docker build -t $DOCKER_HUB_REPO-frontend:latest ./Frontend'
            }
        }

        stage('Push Docker Images') {
            steps {
                echo '☁️ Envoi des images Docker vers Docker Hub'
                withCredentials([usernamePassword(credentialsId: "$DOCKER_HUB_CREDENTIALS_ID", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh 'docker push $DOCKER_HUB_REPO-backend:latest'
                    sh 'docker push $DOCKER_HUB_REPO-frontend:latest'
                }
            }
        }

        stage('Run Docker Compose') {
            steps {
                echo '🚀 Déploiement avec Docker Compose'
                sh 'docker compose down || true'
                sh 'docker compose up -d'
            }
        }
    }

    post {
        failure {
            echo '❌ Échec du pipeline'
        }
        success {
            echo '✅ Pipeline terminé avec succès'
        }
    }
}

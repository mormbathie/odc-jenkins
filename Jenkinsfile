pipeline {
     agent any
 
     environment {
         DOCKER_HUB_CREDENTIALS = 'docker_hub' 
         DOCKERHUB_USER = 'mormbathie'       
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
                         export PATH=$PATH:/var/lib/jenkins/.nvm/versions/node/v22.15.0/bin/
                         npm install
                         npm run build
                        # npm test -- --watchAll=false
                     '''
                 }
             }
         }
 
         stage('Build Docker Images') {
             steps {
                 script {
                     echo "🐳 Construction de l'image Docker Backend"
                     sh "docker build -t ${DOCKERHUB_USER}/odc_backend:latest -f ./Backend/odc/Dockerfile ./Backend/odc"
 
                     echo "🐳 Construction de l'image Docker Frontend"
                     sh "docker build -t ${DOCKERHUB_USER}/odc_frontend:latest ./Frontend"
                 }
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
         stage('run'){
             steps{
                 dir('cd ..'){
                 sh '''
                 docker-compose down || true
                 docker-compose build
                 docker-compose up
                 #docker run --rm -d -p 8081:8081 ${DOCKERHUB_USER}/mon-frontend:latest
                 '''
                 }
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

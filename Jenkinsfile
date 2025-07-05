pipeline {
    agent any
    environment {
        FRONTEND_REPO = 'https://github.com/YassmineTriki/Devops-front'
        BACKEND_REPO = 'https://github.com/YassmineTriki/kaddem.git'
    }
    
    stages {
        stage('Récupération des sources') {
            steps {
                dir('frontend') {
                    git branch: 'master', url: env.FRONTEND_REPO
                }
                dir('backend') {
                    git branch: 'master', url: env.BACKEND_REPO
                }
            }
        }
        
        stage('Déploiement avec Monitoring') {
            steps {
                sh 'docker-compose up -d --build'
            }
        }
    }
}
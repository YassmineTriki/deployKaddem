pipeline {
    agent any
    environment {
        // Référentiels
        FRONTEND_REPO = 'https://github.com/YassmineTriki/front-devops'
        BACKEND_REPO = 'https://github.com/YassmineTriki/kaddemback'
        
        // Variables pour le monitoring
        PROMETHEUS_CONFIG = './monitoring/prometheus.yml'
        GRAFANA_DASHBOARDS = './monitoring/grafana/dashboards'
    }
    
    stages {
        // Étape 1: Récupération des sources
        stage('Récupération des sources') {
            steps {
                dir('frontend') {
                    git branch: 'master', url: env.FRONTEND_REPO
                }
                dir('backend') {
                    git branch: 'master', url: env.BACKEND_REPO
                }
                dir('monitoring') {
                    sh '''
                        mkdir -p grafana/dashboards
                        curl -o prometheus.yml https://raw.githubusercontent.com/yasmine251/monitoring-infra/main/prometheus.yml
                    '''
                }
            }
        }

        // Étape 2: Build des applications
        stage('Build des applications') {
            steps {
                dir('frontend') {
                    sh 'npm install'
                    sh 'npm run build'
                }
                dir('backend') {
                    sh './mvnw clean package'
                }
            }
        }

        // Étape 3: Déploiement avec monitoring
        stage('Déploiement avec Monitoring') {
            steps {
                // Vérification de la config Prometheus
                sh '''
                    docker run --rm -v "${PWD}/monitoring/prometheus.yml:/tmp/prometheus.yml" \
                    prom/prometheus --config.file=/tmp/prometheus.yml --check-config
                '''
                
                // Lancement de l'infrastructure
                sh 'docker-compose -f docker-compose.yml -f monitoring/docker-compose.monitoring.yml up -d --build'
            }
        }

        // Étape 4: Vérification
        stage('Vérification') {
            steps {
                // Vérifier que les services sont up
                sh 'curl --retry 5 --retry-delay 10 --retry-max-time 30 http://localhost:8080/actuator/health'
                sh 'curl --retry 5 --retry-delay 10 --retry-max-time 30 http://localhost:9090/-/healthy'
                
                // Charger les dashboards Grafana (exemple)
                sh '''
                    docker exec grafana \
                    curl -X POST -H "Content-Type: application/json" \
                    -d @/var/lib/grafana/dashboards/spring-boot-dashboard.json \
                    http://admin:admin@localhost:3000/api/dashboards/db
                '''
            }
        }
    }

    post {
        always {
            // Nettoyage optionnel
            sh 'docker-compose down || true'
            cleanWs()
        }
        success {
            slackSend message: "Déploiement réussi avec monitoring - ${env.BUILD_URL}"
        }
        failure {
            slackSend message: "Échec du déploiement - ${env.BUILD_URL}"
        }
    }
}
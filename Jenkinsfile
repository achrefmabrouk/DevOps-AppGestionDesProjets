pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    environment {
        COMPOSE_PROJECT_NAME = 'gestion-projets'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1 --oneline'
            }
        }

        stage('Verifier les outils') {
            steps {
                sh '''
                    docker --version
                    docker compose version
                    docker compose config -q
                '''
            }
        }

        stage('Build des images') { steps { sh ''' echo "=== Début du build Docker ===" 
                                           docker compose build --pull echo "=== Build Docker terminé ===" ''' } }

        stage('Deploiement') {
            steps {
                sh '''
                    docker compose down --remove-orphans
                    docker compose up -d
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    echo "Attente du backend..."
                    for i in $(seq 1 30); do
                        CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8081/entreprise/all || true)
                        if [ "$CODE" = "200" ]; then
                            echo "Backend OK"
                            break
                        fi
                        if [ "$i" = "30" ]; then
                            echo "Le backend ne repond pas"
                            exit 1
                        fi
                        sleep 5
                    done

                    echo "Test du frontend..."
                    curl -sf http://localhost:4200/ > /dev/null
                    echo "Frontend OK"

                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline termine : frontend sur le port 4200, API sur le port 8081.'
        }
        failure {
            echo 'Echec du pipeline, derniers logs :'
            sh 'docker compose logs --tail=50 || true'
        }
    }
}

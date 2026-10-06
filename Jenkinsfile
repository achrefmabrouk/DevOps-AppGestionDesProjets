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

                sh '''
                    echo "=== Checkout terminé ==="
                    git log -1 --oneline
                '''
            }
        }

        stage('Verifier les outils') {
            steps {
                sh '''
                    echo "=== Vérification des outils ==="

                    docker --version
                    docker compose version
                    docker compose config -q

                    echo "=== Tous les outils sont OK ==="
                '''
            }
        }

        stage('Build des images') {
            steps {
                sh '''
                    echo "=== Début du build Docker ==="

                    docker compose build --pull --progress=plain

                    echo "=== Build Docker terminé ==="
                '''
            }
        }

        stage('Deploiement') {
            steps {
                sh '''
                    echo "=== Arrêt des anciens conteneurs ==="

                    docker compose down --remove-orphans

                    echo "=== Démarrage des nouveaux conteneurs ==="

                    docker compose up -d

                    echo "=== Déploiement terminé ==="
                '''
            }
        }

        stage('Verification') {
            steps {
                sh '''
                    echo "=== Vérification du backend ==="
                    echo "Attente du backend..."

                    for i in $(seq 1 30); do

                        CODE=$(curl -s -o /dev/null -w "%{http_code}" \
                            http://localhost:8081/entreprise/all || true)

                        echo "Tentative $i/30 - HTTP Code: $CODE"

                        if [ "$CODE" = "200" ]; then
                            echo "Backend OK"
                            break
                        fi

                        if [ "$i" = "30" ]; then
                            echo "Le backend ne répond pas après 150 secondes."
                            exit 1
                        fi

                        sleep 5
                    done

                    echo "=== Vérification du frontend ==="

                    curl -sf http://localhost:4200/ > /dev/null

                    echo "Frontend OK"

                    echo "=== État des conteneurs ==="

                    docker compose ps
                '''
            }
        }
    }

    post {

        success {
            echo '''
Pipeline terminé avec succès !
Frontend : http://localhost:4200
API      : http://localhost:8081
'''
        }

        failure {
            echo 'Échec du pipeline. Affichage des derniers logs Docker...'

            sh '''
                echo "=== État des conteneurs ==="
                docker compose ps || true

                echo "=== Logs Docker ==="
                docker compose logs --tail=50 || true
            '''
        }
    }
}

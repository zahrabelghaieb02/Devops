pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"                 // Ton registre (ex: Docker Hub)
        IMAGE_NAME = "belghaieb/devops"        // Nom de ton image Docker
        IMAGE_TAG = "latest"                   // Tag de l'image
        DOCKER_CREDENTIALS = 'docker-hub-credentials' // ID des credentials Jenkins pour Docker
    }

    triggers {
        // Webhook GitHub ou fallback avec pollSCM
        pollSCM('* * * * *') // Vérifie chaque minute
    }

    stages {

        stage('Checkout') {
            steps {
                echo "Récupération du dépôt Git..."
                checkout scm
            }
        }

        stage('Clean Workspace') {
            steps {
                echo "Nettoyage du workspace..."
                sh 'git clean -fdx'
            }
        }

        stage('Build Project') {
            steps {
                echo "Reconstruction du projet Maven..."
                sh 'mvn clean package -DskipTests'
            }
        }

        /* ----------------------------- */
        /*     STAGE SONARQUBE           */
        /* ----------------------------- */
        stage('MVN SONARQUBE') {
            steps {
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh """
                        mvn sonar:sonar \
                          -Dsonar.login=${SONAR_TOKEN} \
                          -Dsonar.host.url=http://192.168.x.x:9000 \
                          -Dsonar.projectKey=myproject
                    """
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "Construction de l’image Docker..."
                sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                """
            }
        }

        stage('Push Docker Image') {
            steps {
                echo "Connexion & push sur le registre Docker..."
                withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS,
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin ${REGISTRY}
                        docker push ${IMAGE_NAME}:${IMAGE_TAG}
                        docker logout ${REGISTRY}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Pipeline terminé avec succès 🎉"
        }
        failure {
            echo "Le pipeline a échoué ❌"
        }
    }
}

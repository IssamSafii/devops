pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'issamsafi/backend-app:latest'
    }

    stages {

        stage('Git') {
            steps {
                echo 'Checkout du projet depuis GitHub'
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                echo 'Build Maven du backend'

                dir('backend') {
                    sh 'mvn clean package -DskipTests'
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Construction de image backend-app'

                dir('backend') {
                    sh 'docker build -t backend-app:latest .'
                }
            }
        }

        stage('Docker Push') {
            steps {
                echo 'Publication de image sur Docker Hub'

                withCredentials([
                    usernamePassword(
                        credentialsId: 'aacc046f-afb3-4e7f-ae88-1990550047fa',
                        usernameVariable: 'safi',
                        passwordVariable: '25134109Safi'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_TOKEN" | docker login -u "$DOCKER_USER" --password-stdin

                        docker tag backend-app:latest $DOCKER_IMAGE

                        docker push $DOCKER_IMAGE

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                echo 'Lancement du conteneur MySQL'

                sh '''
                    docker rm -f mysql || true

                    docker network inspect devops-network >/dev/null 2>&1 || \
                    docker network create devops-network

                    docker run -d \
                        --name mysql \
                        --network devops-network \
                        -e MYSQL_ROOT_PASSWORD=root \
                        -e MYSQL_DATABASE=test_db \
                        mysql:8.0

                    echo "Attente du démarrage de MySQL..."

                    until docker exec mysql mysqladmin ping -h localhost -uroot -proot --silent
                    do
                        sleep 5
                    done

                    echo "MySQL est prêt"
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                echo 'Déploiement du backend'

                sh '''
                    docker rm -f backend-app || true

                    docker pull $DOCKER_IMAGE

                    docker run -d \
                        --name backend-app \
                        --network devops-network \
                        -p 8081:8080 \
                        -e SPRING_DATASOURCE_URL="jdbc:mysql://mysql:3306/test_db?createDatabaseIfNotExist=true" \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e SPRING_DATASOURCE_PASSWORD=root \
                        $DOCKER_IMAGE
                '''
            }
        }

        stage('Verification') {
            steps {
                echo 'Vérification du déploiement'

                sh '''
                    sleep 15

                    echo "===== DOCKER PS ====="
                    docker ps

                    echo "===== BACKEND LOGS ====="
                    docker logs backend-app
                '''
            }
        }
    }
}
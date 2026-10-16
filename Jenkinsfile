pipeline {
    agent any

    tools {
        jdk   'JAVA_HOME'
        maven 'M2_HOME'
    }

    environment {
        REGISTRY_USER = 'adembt'
        IMAGE         = "${REGISTRY_USER}/backend-app:latest"
        NETWORK       = 'devops-net'
    }

    stages {

        stage('Git') {
            steps { checkout scm }
        }

        stage('Build & Test Maven') {
            steps { sh 'mvn clean package' }
            post {
                always { junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml' }
            }
        }

        stage('Docker Build') {
            steps { sh 'docker build -t backend-app:latest .' }
        }

        stage('Docker Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub',
                                 usernameVariable: 'DH_USER', passwordVariable: 'DH_PASS')]) {
                    sh '''
                        echo "$DH_PASS" | docker login -u "$DH_USER" --password-stdin
                        docker tag backend-app:latest $IMAGE
                        docker push $IMAGE
                    '''
                }
            }
        }

        stage('Déploiement MySQL') {
            steps {
                sh '''
                    docker network create $NETWORK || true
                    if [ -n "$(docker ps -aq -f name=^mysql$)" ]; then
                        docker start mysql
                    else
                        docker run -d --name mysql --network $NETWORK \
                            -e MYSQL_ROOT_PASSWORD=root \
                            -e MYSQL_DATABASE=test_db \
                            -v mysql-cd-data:/var/lib/mysql \
                            mysql:8.0
                    fi
                    echo "Attente du demarrage de MySQL..."
                    for i in $(seq 1 40); do
                        docker exec mysql mysqladmin ping -h 127.0.0.1 -proot --silent && break
                        sleep 3
                    done
                '''
            }
        }

        stage('Déploiement backend-app') {
            steps {
                sh '''
                    docker stop backend-app || true
                    docker rm backend-app || true
                    docker pull $IMAGE
                    docker run -d --name backend-app --network $NETWORK \
                        --restart unless-stopped \
                        -p 8090:8080 \
                        -e SPRING_DATASOURCE_URL="jdbc:mysql://mysql:3306/test_db?createDatabaseIfNotExist=true" \
                        -e SPRING_DATASOURCE_USERNAME=root \
                        -e SPRING_DATASOURCE_PASSWORD=root \
                        $IMAGE
                '''
            }
        }

        stage('Vérification') {
            steps {
                sh '''
                    for i in $(seq 1 30); do
                        docker logs backend-app 2>&1 | grep -q "Started BackendApplication" && break
                        sleep 3
                    done
                    docker ps
                    docker logs backend-app
                    docker logs backend-app 2>&1 | grep "Started BackendApplication"
                '''
            }
        }
    }

    post {
        always { sh 'docker logout || true' }
    }
}

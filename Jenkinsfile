pipeline {
    agent any
    environment {
        IMAGE_NAME     = "vehiculos-tomcat:1.0"
        CONTAINER_NAME = "vehiculos-app"
        WAR_NAME       = "vehiculosBuild.war"
        TOMCAT_WEBAPPS = "/usr/local/tomcat/webapps"
        APP_PORT       = "9090"
        SPRING_DATASOURCE_URL      = "jdbc:mysql://vehiculo-s8.cluster-chpmtl5ouzig.us-east-1.rds.amazonaws.com:3306/Sucursal?useSSL=false&serverTimezone=UTC&allowPublicKeyRetrieval=true"
        SPRING_DATASOURCE_USERNAME = "admin"
        SPRING_DATASOURCE_PASSWORD = "Admin1234"
    }
    stages {
        stage('Checkout') {
            steps { checkout scm }
        }
        stage('Build WAR (Maven)') {
            steps {
                sh '''
                    chmod +x mvnw
                    ./mvnw -DskipTests clean package
                    ls -lh target/
                '''
            }
        }
        stage('Docker Build (Tomcat Image)') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME} .
                    docker images | head -n 10
                '''
            }
        }
        stage('Deploy Container') {
            steps {
                sh '''
                    docker stop ${CONTAINER_NAME} >/dev/null 2>&1 || true
                    docker rm   ${CONTAINER_NAME} >/dev/null 2>&1 || true
                    docker run -d --name ${CONTAINER_NAME} -p ${APP_PORT}:8080 \
                        -e SPRING_DATASOURCE_URL="${SPRING_DATASOURCE_URL}" \
                        -e SPRING_DATASOURCE_USERNAME="${SPRING_DATASOURCE_USERNAME}" \
                        -e SPRING_DATASOURCE_PASSWORD="${SPRING_DATASOURCE_PASSWORD}" \
                        ${IMAGE_NAME}
                    echo "=== docker ps ==="
                    docker ps
                    docker cp target/${WAR_NAME} ${CONTAINER_NAME}:${TOMCAT_WEBAPPS}/${WAR_NAME}
                    docker restart ${CONTAINER_NAME}
                    echo "=== webapps dentro del contenedor ==="
                    docker exec ${CONTAINER_NAME} ls -lh ${TOMCAT_WEBAPPS} || true
                    docker logs --tail 50 ${CONTAINER_NAME} || true
                '''
            }
        }
        stage('Smoke Test') {
            steps {
                sh '''
                    curl -s http://localhost:${APP_PORT}/vehiculosBuild/ || true
                    curl -I http://localhost:${APP_PORT}/vehiculosBuild/swagger-ui/index.html || true
                '''
            }
        }
    }
    post {
        always {
            archiveArtifacts artifacts: 'target/*.war', fingerprint: true
        }
    }
}
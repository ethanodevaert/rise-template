properties([
    pipelineTriggers([githubPush()])
])

node {
    stage('Checkout') {
        checkout scm
    }

    stage('Build Docker Image') {
        sh 'docker build -t rise-app .'
    }

    stage('Network') {
        sh 'docker network create rise-net || true'
        sh 'docker network connect rise-net postgres_db || true'
    }

    stage('Deploy') {
        sh 'docker rm -f rise-app || true'
        withEnv(['DB_HOST=postgres_db', 'DB_NAME=risedb', 'DB_USER=postgres']) {
            withCredentials([string(credentialsId: 'rise-db-password', variable: 'DB_PASSWORD')]) {
                sh '''
                    docker run -d --name rise-app --network rise-net --restart on-failure:10 \
                      -p 5000:8080 \
                      -e ASPNETCORE_ENVIRONMENT=Development \
                      -e "ConnectionStrings__DefaultConnection=Host=$DB_HOST;Port=5432;Database=$DB_NAME;Username=$DB_USER;Password=$DB_PASSWORD" \
                      rise-app
                '''
            }
        }
    }

    stage('Smoke test') {
        retry(12) {
            sleep 5
            sh 'curl -fsS http://172.17.0.1:5000/ > /dev/null'
        }
    }
}
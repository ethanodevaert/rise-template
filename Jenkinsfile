node {
    stage('Checkout') {
        checkout scm
    }
    
    stage('Preparation') {
        catchError(buildResult: 'SUCCESS') {
            sh 'docker stop rise-app'
            sh 'docker rm rise-app'
        }
    }
    
    stage('Build Docker Image') {
        sh 'docker build -t rise-app .'
    }
    
    stage('Deploy') {
        sh 'docker run -d -p 5000:8080 --name rise-app rise-app'
    }
}
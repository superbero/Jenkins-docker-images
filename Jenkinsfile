pipeline{
    agent any

    stages{
        stage('Checkout Repo 2') {
            steps {
                script {
                    sh 'git clone https://github.com/superbero/Jenkins-namespaces-volumes/'
                    sh ' mv "jenkins-namespaces-volumes" "namespaces_volumes"'
                }
            }
        }
        stage("Checkout Repo 4"){
            steps {
                script {
                    sh 'git clone https://github.com/superbero/Jenkins-namespaces-volumes/'
                    sh ' mv "jenkins-namespaces-volumes" "namespaces_volumes"'
                }
            }
        }
        stage("Checkout Repo 5"){
            steps {
                script {
                    sh 'git clone https://github.com/superbero/Jenkins-movie-service/'
                    sh ' mv "Jenkins-movie-service" "movie-service"'
                }
            }
        }
        stage("Checkout Repo 6"){
            steps {
                script {
                    sh 'git clone https://github.com/superbero/Jenkins-cast-service/'
                    sh ' mv "Jenkins-cast-service" "cast-service"'
                }
            }
        }
         stage("Checkout Repo 7"){
            steps {
                script {
                    sh 'git clone https://github.com/superbero/Jenkins-api-service/'
                    sh ' mv "Jenkins-api-service" "api-service"'
                }
            }
        }
        stage("Login to docker hub"){
            steps{
                script {
                    sh "echo 'login to docker hub'"
                    // sh 'docker --version'
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                        sh '''
                        $docker --version
                        $docker login -u $DOCKER_HUB_USERNAME -p $DOCKER_HUB_PASSWORD
                        '''
                        }
                    }
            }
        }
        stage("build docker images") {
            steps {
                sh '''
                cd docker-images/
                echo "building movie-service image"
                $docker build . -t superbero/movie_service:latest
                echo "building cast-service image"
                $docker build . -t superbero/cast_service:latest
                '''
            }
        }
        stage("push to docker hub") {
            steps {
                sh '''
                $docker push superbero/movie_service:latest
                $docker push superbero/cast_service:latest
                '''
            }
        }
        stage ("Deploy volumes"){
            steps {
                sh '''
                $kubectl apply -f namespaces_volumes/namespaces
                '''
            }
        }
        stage ("Deploy namespaces"){
            steps {
                sh '''
                $kubectl apply -f namespaces_volumes/namespaces/
                '''
            }
        }
    }
}

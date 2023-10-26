pipeline{
    agent any

    stages{
        stage('Checkout scm') {
            steps {
                script {
                    sh '''
                    git clone https://github.com/superbero/Jenkins-namespaces-volumes/
                    mv "jenkins-namespaces-volumes" "namespaces_volumes"
                    git clone https://github.com/superbero/Jenkins-movie-service/
                    mv "Jenkins-movie-service" "movie-service"
                    git clone https://github.com/superbero/Jenkins-cast-service/
                    mv "Jenkins-cast-service" "cast-service"
                    git clone https://github.com/superbero/Jenkins-api-service/
                    mv "Jenkins-api-service" "api-service"
                    '''
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
        stage("Deploy movie-service"){
            steps{
                script{
                    def namespaces = ['dev', 'staging', 'prod', 'qa']
                        namespaces.each { namespace ->
                        echo "Deploying ${namespace} node"
                        try 
                        {
                            sh "sed -i.bak 's/namespace: dev/namespace: ${namespace} /g' movie-service/values.yaml"
                            sh "$helm install jenkins-movie-service movie-service/ --values=movie-service/values.yaml -n ${namespace}"
                            sh "sed -i.bak 's/namespace: ${namespace}/namespace: dev /g' movie-service/values.yaml"
            
                        } catch(Exception e)
                        {
                            echo "Namespace ${namespace} not found, creating..."
                            currentBuild.result = 'UNSTABLE' // Set build result to UNSTABLE
                            sh "$kubectl get all -n ${namespace}"
                        }
                    }
                
                }
            }
        }
        stage("Deploying cast-service"){
            steps{
                script{
                    echo "Deploying cast service"
                }
            }
        }
    }
}

pipeline{
    agent any

    stages{
        stage ("pre-build")
        {

            steps{
                //clone first repo
                checkout([$class: 'MultiSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/superbero/Jenkins-docker-images.git']]])
                //clone second repo
                checkout([$class: 'MultiSCM', branches: [[name: '*/master']], userRemoteConfigs: [[url: 'https://github.com/superbero/Jenkins-namespaces-volumes.git']]])

                // git (url: 'https://github.com/superbero/Jenkins-namespaces-volumes.git', branch: 'master' )
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
    }
}

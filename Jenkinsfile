pipeline{
    agent any

    stages{
        stage('Checkout scm') {
            steps {
                script {
                    sh '''
                    git clone https://github.com/superbero/Jenkins-namespaces-volumes/
                    mv "jenkins-namespaces-volumes" "namespaces_volumes"
                    git clone https://github.com/superbero/Jenkins-database-service.git
                    mv "Jenkins-database-service" "databases"
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
                $docker build movie-service/. -t superbero/movie_service:latest
                echo "building cast-service image"
                $docker build cast-service/. -t superbero/cast_service:latest
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
        stage('Select Service') {
            steps {
                script {
                    def userInput = input(
                        id: 'userInput',
                        message: 'Select the service to update:',
                        parameters: [
                            choice(name: 'Service', choices: 'New Deployment\nService Upgrade', description: 'Select the service')
                        ]
                    )
                    env.SERVICE_NAME = userInput
                    echo env.SERVICE_NAME
                }
            }
        }
        stage ("Deploy volumes"){
            when{
                expression { env.SERVICE_NAME == 'New Deployment'} 
            }
            steps {
                sh '''
                $kubectl apply -f namespaces_volumes/volumes/
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
        
        stage("Deploy database") {
            when{
                expression { env.SERVICE_NAME == 'New Deployment'} 
            }
            steps{
                script{
                    def namespaces = ['dev', 'staging', 'qa']
                        namespaces.each { namespace ->
                        echo "Deploying ${namespace} node"
                        try 
                        {
                            sh "sed -i.bak 's/namespace: dev/namespace: ${namespace} /g' databases/postgres/values.yaml"
                            sh "$helm install jenkins-database-service databases/postgres/ --values=databases/postgres/values.yaml -n ${namespace}"
                            sh "sed -i.bak 's/namespace: ${namespace}/namespace: dev /g' databases/postgres/values.yaml"
                            sh "$kubectl get all -n ${namespace}"
            
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
        stage("Deploy movie-service"){
            when{
                expression { env.SERVICE_NAME == 'New Deployment'} 
            }
            steps{
                script{
                    sleep(60)
                    def namespaces = ['dev', 'staging', 'qa']
                        namespaces.each { namespace ->
                        echo "Deploying ${namespace} node"
                        try 
                        {
                            sh "sed -i.bak 's/namespace: dev/namespace: ${namespace} /g' movie-service/values.yaml"
                            sh "$helm install jenkins-movie-service movie-service/ --values=movie-service/values.yaml -n ${namespace}"
                            sh "sed -i.bak 's/namespace: ${namespace}/namespace: dev /g' movie-service/values.yaml"
                            sh "$kubectl get all -n ${namespace}"
            
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
            when{
                expression { env.SERVICE_NAME == 'New Deployment'} 
            }
            steps{
                script{
                    def namespaces = ['dev', 'staging', 'qa']
                        namespaces.each { namespace ->
                        echo "Deploying ${namespace} node"
                        try 
                        {
                            sh "sed -i.bak 's/namespace: dev/namespace: ${namespace} /g' cast-service/values.yaml"
                            sh "$helm install jenkins-cast-service cast-service/ --values=cast-service/values.yaml -n ${namespace}"
                            sh "sed -i.bak 's/namespace: ${namespace}/namespace: dev /g' cast-service/values.yaml"
                            sh "$kubectl get all -n ${namespace}"
            
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
        stage("Deploying api-service"){
            when{
                expression { env.SERVICE_NAME == 'New Deployment'} 
            }
            steps{
                script{
                    def namespaces = ['dev', 'staging', 'qa']
                        namespaces.each { namespace ->
                        echo "Deploying ${namespace} node"
                        try 
                        {
                            sh "sed -i.bak 's/namespace: dev/namespace: ${namespace} /g' api-service/values.yaml"
                            sh "$helm install jenkins-api-service api-service/ --values=api-service/values.yaml -n ${namespace}"
                            sh "sed -i.bak 's/namespace: ${namespace}/namespace: dev /g' api-service/values.yaml"
                            sh "$kubectl get all -n ${namespace}"
            
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
        stage('Update Service') {
            when {
                expression { env.SERVICE_NAME != 'New Deployment' }
            }
            steps {
                script {
                    def userInput = input(
                        id: 'userInput',
                        message: 'Select the service to update:',
                        parameters: [
                            choice(name: 'Service', choices: 'Upgrade api-service\nUpgrade database\nUpgrade movie-service\nUpgrade cast-service', description: 'Select the service you want to upgrade:')
                        ]
                    )
                    env.SERVICE_UPGRADE = userInput
                    if (eenv.SERVICE_UPGRADE == 'Upgrade api-service') {
                        sh '''
                        $helm upgrade jenkins-api-service . --values=values.yaml -n dev
                        $helm upgrade jenkins-api-service . --values=values.yaml -n qa
                        $helm upgrade jenkins-api-service . --values=values.yaml -n staging
                        '''
                        // Add steps to update service1
                    } else if (env.SERVICE_UPGRADE == 'Upgrade database') {
                        sh '''
                        $helm upgrade jenkins-database-service . --values=values.yaml -n dev
                        $helm upgrade jenkins-database-service . --values=values.yaml -n qa
                        $helm upgrade jenkins-database-service . --values=values.yaml -n staging
                        '''
                        // Add steps to update service2
                    } else if (env.SERVICE_UPGRADE == 'Upgrade cast-service') {
                        // Add steps to update service2
                        sh '''
                        $helm upgrade jenkins-cast-service . --values=values.yaml -n dev
                        $helm upgrade jenkins-cast-service . --values=values.yaml -n qa
                        $helm upgrade jenkins-cast-service . --values=values.yaml -n staging
                        '''
                    } else if (env.SERVICE_UPGRADE == 'Upgrade movie-service') {
                        // Add steps to update service3
                        sh'''
                        $helm upgrade jenkins-movie-service . --values=values.yaml -n dev
                        $helm upgrade jenkins-movie-service . --values=values.yaml -n qa
                        $helm upgrade jenkins-movie-service . --values=values.yaml -n staging
                        '''
                    } else {
                        error 'Invalid service selected'
                    }
                }
            }
        }
    }
}

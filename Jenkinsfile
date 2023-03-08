pipeline
{
    agent
    {
        label 'docker-node'
    }
    environment
    {
        DOCKERHUB_CREDENTIALS=credentials('docker-hub')
    }
    stages
    {
        stage('Clone')
        {
            steps
            {
                git branch: 'master', url: 'http://192.168.99.102:3000/veneta/bgapp'
            }  
        }
        stage('Create a network')
        {
            steps
            {
                sh '''
                docker network rm appnet pipeline-bgapp_appnet deploy_appnet || true
                docker network ls | grep appnet || docker network create appnet
                '''
            }
        }
        stage('Build the Images')
        {
            steps
            {
                sh 'docker image rm -f pipeline-bgapp-web pipeline-bgapp-db || true'
                sh 'docker compose build'
            }
        }
        stage('Run the Containers')
        {
            steps
            {
                sh 'docker container rm -f deploy-web-1 deploy-db-1 || true'
                sh 'docker volume prune -f'
                sh 'docker compose up -d'
            }
        }
        stage('Test')
        {
            steps
            {
                script
                {
                    echo 'Test #1 - Reachability'
                    sh 'echo $(curl --write-out "%{http_code}" --silent --output /dev/null http://192.168.99.102:8080) | grep 200 || true'

                    echo 'Test #2 - Check if Plovdiv Appears'
                    sh "curl --silent http://192.168.99.102:8080 | grep Пловдив || true"
                }
            }
        }
        stage('CleanUp')
        {
            steps
            {
                sh 'docker container rm -f pipeline-bgapp-web-1 pipeline-bgapp-db-1 || true'
                sh 'docker volume prune -f'
            }
        }
        stage('Login')
        {
            steps
            {
                sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
            }
        }
        stage('Push')
        {
            steps
            {
                sh 'docker image rm loradurst/bgapp-web loradurst/bgapp-db || true'
                sh 'docker image tag pipeline-bgapp-web loradurst/bgapp-web'
                sh 'docker image tag pipeline-bgapp-db loradurst/bgapp-db'
                sh 'docker push loradurst/bgapp-web'
                sh 'docker push loradurst/bgapp-db'
            }
        }
        stage('Deploy')
        {
            steps
            {
                sh 'docker image rm -f pipeline-bgapp-web pipeline-bgapp-db || true'
                sh 'cd deploy'
                sh 'docker compose up -d'
            }
        }
    }
}
pipeline{
    agent{label 'worker'}
    stages{
        stage("Docker build and push"){
            steps{
                sh "docker login -u sagardpatil0055 -p 009D@12345"
                sh '''
                    cd vote
                    docker build -t sagardpatil0055/vote:v${BUILD_NUMBER} .
                    '''
                sh "docker push sagardpatil0055/vote:v${BUILD_NUMBER}"
            }
        }
        stage("Deploy"){
            steps{
                sh "echo docker deploy"
            }
        }
    }
}

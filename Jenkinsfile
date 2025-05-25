pipeline{
    agent{label 'worker'}
    options { 
        buildDiscarder(logRotator(numToKeepStr: '15'))
        disableConcurrentBuilds()
        retry(2)
        timeout(time: 10, unit: 'MINUTES')
    }
    parameters {
        string(name: 'BRNACH', defaultValue: 'develop', description: '')
        booleanParam(name: 'TEST_CASES', defaultValue: true, description: '')
        choice(name: 'ENV', choices: ['dev', 'qa', 'uat'], description: '')
    }
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
        stage ("parallel testing"){
            parallel{    
               	stage("Linux Test"){
			agent{label 'worker'}
                    when { branch 'develop'
                	    environment name: 'DEPLOY_TO', value: 'qa' 
			 }
		  	     
                   	 steps{sh "echo linux"
                      		sh "sleep 180" }
               		 }  
               		 stage("Windows Test"){
				 when {		 branch 'develop'
                	  		 	 environment name: 'DEPLOY_TO', value: 'qa' 
					 }
                   		 agent{label 'worker'}
                   		 steps{
                       			 sh "echo windows"
                       			 sh "sleep 180"
                    }
                }                       
        }
    }
}

}

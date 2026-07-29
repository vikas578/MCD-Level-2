pipeline {

    agent any
    
     options {
	    skipDefaultCheckout(true)
	}
    
    
    parameters {
	    choice(
	        name: 'ENV',
	        choices: ['dev', 'qa', 'prod'],
	        description: 'Select deployment environment'
	    )
	}


    stages {
    	
    	stage('Checkout') {
		    steps {
		    	cleanWs()
		        checkout scm
		    }
		}

        stage('Build') {
            steps {
				bat "mvn clean package -Denv=${params.ENV}"
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }


        stage('Deploy to CH2.0') {
            steps {

                withCredentials([
                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.USER',
                           variable: 'ANYPOINT_CONNECTED_APP_USER'),
                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.PASSWORD',
                           variable: 'ANYPOINT_CONNECTED_APP_PASSWORD'),
                    string(credentialsId: 'ANYPOINT.USERNAME',
                           variable: 'ANYPOINT_USERNAME'),
                    string(credentialsId: 'ANYPOINT.PASSWORD',
                           variable: 'ANYPOINT_PASSWORD')
                ]) {

                    configFileProvider([
                        configFile(
                            fileId: 'mulesoft-settings',
                            variable: 'MAVEN_SETTINGS'
                        )
                    ]){

                      	//bat "mvn -s \"${env.MAVEN_SETTINGS}\" deploy -DmuleDeploy -Denv=${params.ENV}"

						bat "mvn -s \"${env.MAVEN_SETTINGS}\" mule:deploy -Denv=${params.ENV}"	
                    }
            	}
            }
        }

    }   // <-- End of stages

    post {

        success {
            echo "Deployment to ${params.ENV} completed successfully."
        }

        failure {
            echo "Deployment to ${params.ENV} failed."
        }

        always {
            cleanWs()
        }

    }

}       // <-- End of pipeline
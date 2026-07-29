pipeline {

    agent any
    
     options {
	    skipDefaultCheckout(true)
	}
    
    
    parameters {
	    choice(
	        name: 'ENV',
	        choices: ['dev', 'qa', 'prod'],
	        description: 'Select properties file'
	    )
	}


    stages {
    	
    	stage('Checkout') {
		    steps {
		        cleanWs()
		        checkout scm
		    }
		}


		stage('Verify Git Revision') {
            steps {
                bat 'git log -1 --oneline'
            }
        }
        
        
        stage('Build & Publish to Exchange') {
            steps {

                withCredentials([
                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.USER',
                           variable: 'ANYPOINT_CONNECTED_APP_USER'),
                    string(credentialsId: 'ANYPOINT.CONNECTED.APP.PASSWORD',
                           variable: 'ANYPOINT_CONNECTED_APP_PASSWORD')
                ]) {

                    configFileProvider([
                        configFile(
                            fileId: 'mulesoft-settings',
                            variable: 'MAVEN_SETTINGS'
                        )
                    ]) {

                        bat """
                        mvn -B ^
                        -s "%MAVEN_SETTINGS%" ^
                        clean deploy ^
                        -Denv=${params.ENV}
                        """

                    }
                }
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
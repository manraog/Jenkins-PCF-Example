pipeline{
 	
 	agent { label 'java11' }

  	stages{
  		stage ('Build') {
  			steps {	
	  			office365ConnectorSend(
	  				message: "Pipeline iniciado ", 
	  				status:"Started", 
	  				webhookUrl:'https://outlook.office.com/webhook/af014299-a5b2-4be6-af45-b6b7b6f19051@ac5349df-152e-486f-9b39-fe3c4a25efe0/JenkinsCI/490b65bc07794f61954227e050d9d1e8/10ed5eda-9b70-4598-858a-e5ae6599fa66'
	  			)
	  			sh 'echo $JAVA_HOME'
	  			sh 'ls -la /usr/lib/jvm/'
	  			sh './gradlew bootJAr --no-daemon'
	  		}
        }

        stage('Test') {
        	steps{
        		sh './gradlew test --no-daemon'
        		publishHTML([
        			reportDir: 'build/reports/tests/test',
        			reportFiles: 'index.html',
        			reportName: 'Tests Report',
        			keepAll: true,
					allowMissing: false, 
					alwaysLinkToLastBuild: false
        		])
        	}
        }

    }
}
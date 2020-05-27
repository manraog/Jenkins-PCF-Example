pipeline{
 	
 	agent any

  	stages{
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

  		stage ('Package') {
  			steps {	
	  			sh './gradlew bootJAr --no-daemon'
	  		}
        }

        stage ('CF Push'){
        	steps {
	        	pushToCloudFoundry(
				  target: 'api.run.pivotal.io',
				  organization: 'joel.hernandez-org',
				  cloudSpace: 'development',
				  credentialsId: 'pcfdev_user',
				  manifestChoice: [manifestFile: 'manifest.yml']
				)
        	}
        }
    }
}
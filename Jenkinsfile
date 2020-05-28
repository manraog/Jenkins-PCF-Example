pipeline{
 	
 	agent any

  	stages{
        stage('Test') {
        	steps{
        		withGradle {
        			sh './gradlew test'
        		}
        	}
        }

  		stage ('Package') {
  			steps {	
	  			withGradle {
	  				sh './gradlew bootJar'
	  			}
	  		}
        }

        stage ('CF Push'){
        	when { branch 'master' }
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

    post {
        always {
            archiveArtifacts artifacts: 'build/libs/*.jar', fingerprint: true
            junit 'build/test-results/test/*.xml'
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
pipeline
{
  // This pipeline requires the following plugins:
  // * Slack Notification 
  // * docker...
    agent { label 'agent1_for_website' }
    environment {
                      dockerhub=credentials('dockerhub')
		}

    stages {
        stage('Prune agent data')  {
                steps {			
                           sh 'sudo docker system prune -a -f'
                      }
                                   }

        stage('Build docker images') {
                steps {
			/* set nginx from existing image:
			   sh 'echo $dockerhub_PSW | sudo docker login -u $dockerhub_USR --password-stdin'
                           sh 'sudo docker pull doronamsalem/website:nginx'
			*/
			   sh 'sudo docker-compose build'
		}
	}

        stage('Test') {
                steps {
			    echo "****building containers for testing*****"
                            sh '''
                                sudo docker-compose up -d
                                sudo docker-compose ps
                                '''
			    sh 'sleep 5'
                            sh 'python3 ./unit_tests/website_unittest.py'
                            // sh 'python3 ./unit_tests/api_response_unittest.py'
                }
	}

        stage('Deliver images to DockerHub') {
                steps {
                            // Loging to DockerHub
                            sh 'echo $dockerhub_PSW | sudo docker login -u $dockerhub_USR --password-stdin'
			    sh '''
				sudo docker push doronamsalem/website:nginx 
				sudo docker push doronamsalem/website:webserver
			    '''
			    sh 'sudo docker logout'
                }
         }

        stage('Deploy on EC2') {
                steps {
			sh '''
				scp -i "/home/ubuntu/.ssh/key_for_production" ./production/docker-compose.yml ubuntu@172.31.92.213:/home/ubuntu
				ssh -i "/home/ubuntu/.ssh/key_for_production" -T ubuntu@172.31.92.213 "
					echo $dockerhub_PSW | sudo docker login -u $dockerhub_USR --password-stdin
					sudo docker pull doronamsalem/website:nginx
					sudo docker pull doronamsalem/website:webserver
					sudo docker-compose --file /home/ubuntu/docker-compose.yml down --remove-orphans -v
					sudo docker-compose --file /home/ubuntu/docker-compose.yml up -d
					sudo docker-compose ps
					sudo docker logout
					exit
					"
				'''
		}	
	}
    }
    post {
	    always {
                         sh 'sudo docker-compose down --remove-orphans -v'
                         sh 'echo "removed all containers and images"'
			 deleteDir()
            }

            success {    
                         slackSend channel: '#succeeded-build',
                         color: 'good',
                         message: "${currentBuild.fullDisplayName} completed successfully."
            }       
                            
            failure {
                         slackSend channel: '#devops-alerts',
                         color: 'danger',
                         message: "${currentBuild.fullDisplayName} FAILED"
            }
     }
}

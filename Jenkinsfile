properties([pipelineTriggers([githubPush()])])

pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

	environment {
		PYTHONPATH = "${WORKSPACE}" 
		MY_PROD_CRED = credentials('ssh-prod')
		registry = "petrovskyipavlo/wordpress" 
        registryCredential = 'dockerhub' 
        dockerImage = '' 

	}

    stages {

		
		stage("Checkout") {			 
            steps {
                git credentialsId: 'github-key', url: 'https://github.com/petrovskyipavlo/final_task_iac.git', branch: 'main' 
            }
	    } 
		
		

        
        stage("Build") {
            steps { 
				
				script {
					dockerImage=docker.build registry + ":$BUILD_NUMBER"
				}
			}
		}

		stage("Deploy image on dockerhub")
		{
			steps{
				script{
					docker.withRegistry('', registryCredential) {
						dockerImage.push()
					}
				}
			}
		}

						

        //Manual confirmation of application deployment on the server 
        stage("Approve") {
            steps { approve('Do you want to deploy to production?') }
		}

		stage("Clean up  Jenkins master")
		{
			steps{
				sh  '''#!/bin/bash
				       docker stop $(docker ps -a -q) 2> /dev/null || true
					   docker rm -f $(docker ps -a -q) 2> /dev/null || true
					   docker rmi -f $(docker images -a -q) 2> /dev/null || true
					   yes | docker system prune -f 2> /dev/null || true
					'''				

			}
			
		}
		      		
			
		stage("Clean up deployment server")
		{
			steps{
				
				sshagent(credentials : ['ssh-aws']) {
					sh  '''#!/bin/bash					                   
				           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker stop $(docker ps -a -q) 2> /dev/null || true'
                           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker rm -f $(docker ps -a -q) 2> /dev/null || true'
	                       ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker rmi -f $(docker images -a -q) 2> /dev/null || true'				    
				           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'yes | docker system prune -f 2> /dev/null || true'
				        '''	
				
			    }
			}
			
		}
        

       stage("Pull image from dockerhub")
		{
			steps{
                sshagent(credentials : ['ssh-aws']) {                    
					
					
					sh "ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} uptime"
                    sh "ssh -v ubuntu@${SERVER_IP}"
					sh "ssh  ubuntu@${SERVER_IP} docker pull ${registry}:${BUILD_NUMBER}"
                }
            }
		}

		stage ("Run Wordpress") {
            steps{
                sshagent(credentials : ['ssh-aws']) {                    
					
					sh "ssh -o StrictHostKeyChecking=no ubuntu@${SERVER_IP} uptime"
                    sh "ssh -v ubuntu@${SERVER_IP}"					
					sh "ssh  ubuntu@${SERVER_IP}  docker run -d -p 8080:5000 ${registry}:${BUILD_NUMBER}"
					
                }
            }
        }
    

	    stage("Approve stop application and cleanup server") {
            steps { approve('Do you want to destroy application?') }
		}

		stage("Clean up prod after stop Wordpress ")
		{
			steps{
				
				sshagent(credentials : ['ssh-prod']) {
					sh  '''#!/bin/bash					                   
				           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker stop $(docker ps -a -q) 2> /dev/null || true'
                           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker rm -f $(docker ps -a -q) 2> /dev/null || true'
	                       ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'docker rmi -f $(docker images -a -q) 2> /dev/null || true'				    
				           ssh -o "StrictHostKeyChecking=no" ubuntu@${SERVER_IP} 'yes | docker system prune -f 2> /dev/null || true'
                           
				        '''	
				
			    }
			}
			
		}
 
	}
}





def approve(msg) {

	timeout(time:1, unit:'DAYS') {
		input(msg)     
	}
}



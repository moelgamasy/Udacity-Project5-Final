pipeline {
	agent any

   	environment {
        DOCKER_IMAGE_NAME = "mohamedelgaamsy/capstoneimage"
        docker_hub_login = "mohamedelgaamsy"
        RegCred = 'docker-hub'
	}

	stages {

		stage('Lint HTML') {
			steps {
				sh 'tidy -q -e *.html'
			}
		} 
                

                 stage('Set K8S Context') {
                       steps {
                              withAWS(credentials:'mohamed'){
                             sh "kubectl config set-context arn:aws:eks:us-west-2:902919507921:cluster/EKS-Infra"
                       }
                  }
        }
		
		stage('Build Docker Image') {
            		steps {
                		script {
                    			app = docker.build(DOCKER_IMAGE_NAME)
                    		
                		}
            		}

		}

       		   stage('Push Image to Docker-Hub'){
            steps{
                script{
                    docker.withRegistry('', RegCred){
                        app.push()
                    }
                }
            }
        }




                 stage('Deploy blue & Green container') {
            		steps {
                          sshagent(['Project']) {
                             sh "scp -o StrictHostKeyChecking=no  blue-controller.yaml green-controller.yaml blue-service.yaml cloud@90.84.178.188:/home/cloud/"
                            script{
                             try{
                             sh "ssh cloud@90.84.178.188 kubectl apply -f ."
                             }catch(error){
                             sh "ssh cloud@90.84.178.188 kubectl create -f ."
                                 }
                                 }
                              }
                            }
                         }



        
          

		stage('Approval Stage') {
            steps {
                input "Ready to switch the traffic to green?"
            }
        }




                 stage('Green Deployment') {
                        steps {
                          sshagent(['Project']) {
                           sh "scp -o StrictHostKeyChecking=no  green-service.yaml cloud@90.84.178.188:/home/cloud/run/"
                           script{
                             try{

                             sh "ssh cloud@90.84.178.188 kubectl apply -f /home/cloud/run/green-service.yaml"
                             }catch(error){
                             sh "ssh cloud@90.84.178.188 kubectl create -f /home/cloud/run/green-service.yaml"
                             }
                             }
                                          }
                            }
                         }

}
}

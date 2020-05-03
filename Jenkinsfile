def stage_actual = []					



properties([
    parameters([
        [
            $class: 'ChoiceParameter', 
            choiceType: 'PT_SINGLE_SELECT', 
            description: 'Seleccione la rama de GIT para descargar', 
            filterable: false,
            name: 'NOMBRERAMA',
            randomName: 'NOMBRERAMA',
			randomName: 'choice-parameter-5631314439613978',			
            script: [			
                $class: 'GroovyScript', 
                fallbackScript: [
                    classpath: [], 
                    sandbox: false, 
                    script: 
                        'return[\'No se puede consultar la Rama\']'
                ],
                script: [
                    classpath: [], 
                    sandbox: false, 
                    script: 
                         '''  def gettags = ("git ls-remote -t -h https://github.com/nat55555/timeoff-management-application.git").execute()   
							return gettags.text.readLines().collect { it.split()[1].replaceAll('refs/heads/', '').replaceAll('refs/tags/', '').replaceAll("\\\\^\\\\{\\\\}", '')} '''
                ]				
                
            ]
        ]
    ])
])					
					
pipeline {
    agent any
	options{
    buildDiscarder(logRotator(numToKeepStr: '30'))
    disableConcurrentBuilds()
    }
	triggers {
		pollSCM('* * * * *')
	}
	environment {
		TARGET_SERVER="192.168.86.86"
	}

	stages{


        stage('checkout from git') { 
            steps { 
				script{ 
												try {

															updateGitlabCommitStatus name: 'build', state: 'pending'
															
															echo " la rama gitlabBranch ---> ${env.gitlabBranch}"
															
															if ( env.gitlabBranch != null) {
															NOMBRERAMA=env.gitlabBranch
															}
															
															echo " la rama NOMBRERAMA ---> ${NOMBRERAMA}"	

											

														checkout([$class: 'GitSCM',
														branches: [[name: '*/${NOMBRERAMA}']],
														doGenerateSubmoduleConfigurations: false,
														extensions: [[$class: 'CleanCheckout',  ]],
														submoduleCfg: [],
														userRemoteConfigs: [[url: 'https://github.com/nat55555/timeoff-management-application.git']]])	
														
										


												} catch (e) {
													echo('detected failure: checkout de git')
													stage_actual.add(env.STAGE_NAME)
													throw(e)
										 		}	
				}
			}
        } //fin checkout	


    stage('Docker image build') {
            steps { 
				script{ 
							try {

										//in this step we build the app based on the provided dockerfile
	
										//app = docker.build("timeoff-management-application"+":$BUILD_NUMBER")
										sh "/usr/local/bin/docker-compose build"						

							} catch (e) {
								echo('detected failure: Docker Image build')
								stage_actual.add(env.STAGE_NAME)
								throw(e)
							}	
				}
			}	
	

    }
		
		

    //stage('Test image') {

    //}
	
    stage('Docker image run') {
            steps { 
				script{ 
							try {
								
								sh "/usr/local/bin/docker-compose rm -f "	
							//	sh "docker run -d -p 3000:3000 --name docker-tom  'timeoff-management-application:${BUILD_NUMBER}'"
								sh "/usr/local/bin/docker-compose up -d"
								sh "docker ps"	
								
							} catch (e) {
								echo('detected failure: Docker Image run')
								stage_actual.add(env.STAGE_NAME)
								throw(e)
							}	
				}
			}	
	
    } //Docker Image run		
		
         stage('Push docker Image to Nexus') { 
            steps { 
				script{ 
												try {

														stage('Push Docker Images to Nexus Registry'){
														sh "echo changeme | docker login -u admin --password-stdin 192.168.86.86:8081"
														sh "/usr/local/bin/docker-compose push 192.168.86.86:8081/'timeoff-management-application:'+${BUILD_NUMBER}}"
														//sh "docker rmi $(docker images --filter=reference="NexusDockerRegistryUrl/ImageName*" -q)'"
														sh "docker logout NexusDockerRegistryUrl"
														}

												} catch (e) {
													echo('detected failure: Push docker Image to Nexus')
													stage_actual.add(env.STAGE_NAME)
													throw(e)
										 		}	
				}
			}
        } //fin Push docker Image to Nexus



       
        stage('Reports') {
            steps { 
				script{ 
												try {
	

														//	junit '**/surefire-reports/*.xml'
														
												
														
														//	dependencyCheckPublisher pattern: 'aaa/**/target/dependency-check-report.xml'


			 
												} catch (e) {
													echo('detected failure: Reports')
													stage_actual.add(env.STAGE_NAME)
													throw(e)
										 		}
						} // fin script
					
			} //fin steps
		} // fin reports
		
	} // fin stages

	/* TODO: Setup slack notification
		post {
			always {
				script{ 				 
												try {
																			
														wrap([$class: 'BuildUser']) {
														USUARIODELBUILD="${BUILD_USER}"
														}
														if ( USUARIODELBUILD == "SCMTrigger" ) {
															USUARIODELBUILD=USUARIODELBUILD + " - " + env.gitlabUserName
														}
														
														
														String status
														switch (currentBuild.currentResult) {
															case 'SUCCESS':
																status = 'success'
																echo "POST ACTIONS ---------->"
																
																break
															case ['UNSTABLE', 'FAILURE']:
																status = 'failed'
																break
															default:
																status = 'failed'
														}
														updateGitlabCommitStatus ( name: 'build', state: status	)	
														
												} catch (e) {
													echo('------>>> detected failure: set variables para notificacion')
													stage_actual.add(env.STAGE_NAME)
													throw(e)
										 		}	
				
				}			
			
			}
				
			failure {			
				notifyFailed("${stage_actual}")
			}
			success{
					script{
                             
								notifySuccessful()
					}
			}
			unstable{
					script{
			
					
						notifyUnstable()
					}
			}
		} */



} // fin pipeline

def notifyFailed(nombre_stage_actual) {

  slackSend (color: '#FF0000', message: "Fallo en la ejecución de ésta tarea: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})  \n *Job ejecutado por* ${USUARIODELBUILD} ")
}



def notifySuccessful() {
  slackSend (color: '#00FF00', message: "Tarea finalizada exitosamente: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) \n *Job ejecutado por* ${USUARIODELBUILD} \n *El despliegue se hizo a partir de la rama:* ${NOMBRERAMA}  ")
}

def notifyUnstable() {
  slackSend (color: '#FFFF00', message: "Tarea finalizada con resultado Inestable: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) \n *Job ejecutado por* ${USUARIODELBUILD} \n *El análisis se hizo a partir de la rama:* ${NOMBRERAMA}   ")
}


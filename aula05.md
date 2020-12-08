[TOC]

------

# Aula 05

## Pipeline

### Adicionando aprovação

```
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/jlosantana/vuttr.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -DskipTests clean package'
            }
        }
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        // Importante observar o Dockerfile disponível na aplicação
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${docker_image} .'
            }
        }
        // Para qualquer execução do Container em andamento
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f ${docker_container}'
                    } catch (e) {}
                }
            }
        }
        // Executa a Container da aplicação
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name ${docker_container} -d -p 3000:3000 ${docker_image}'
            }
        }
        stage ('User Approval') {
        	steps {
        		script {
        			slackSend (color: 'warning', message: "Delpoy aguardando aprovação: ${JOB_URL}", tokenCredentialId: 'Slack')
        			timeout(time: 10, unit: 'MINUTES') {
        				input(id: "Deploy Gate", message: "Deploy em homologação?", ok: 'Deploy')
        			}
        		}
        	}
        }
    }
}
```

### Chamando outro Job 

Primeiramente definimos um novo Job que recebe como parêmetro os seguintes valores:

* docker_image: Nome da imagem docker de homologação. Valor padrão jlosantana/vuttr-hml.

* docker_container: Nome padrão de execução do container de homologação. Valor padrão vuttr-hml.

```
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'hml', url: 'https://github.com/jlosantana/vuttr.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -DskipTests clean package'
            }
        }
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        // Importante observar o Dockerfile disponível na aplicação
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${docker_image} .'
            }
        }
        // Para qualquer execução do Container em andamento
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f ${docker_container}'
                    } catch (e) {}
                }
            }
        }
        // Executa a Container da aplicação
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name ${docker_container} -d -p 3001:3000 ${docker_image}'
            }
        }
    }
}
```

Assim podemos alterar o job anterior para chamar nosso novo job.

```
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/jlosantana/vuttr.git'
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -DskipTests clean package'
            }
        }
        stage('Tests') {
            steps {
                sh 'mvn test'
            }
        }
        // Importante observar o Dockerfile disponível na aplicação
        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${docker_image} .'
            }
        }
        // Para qualquer execução do Container em andamento
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f ${docker_container}'
                    } catch (e) {}
                }
            }
        }
        // Executa a Container da aplicação
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name ${docker_container} -d -p 3000:3000 ${docker_image}'
            }
        }
        stage ('User Approval') {
        	steps {
        		script {
        			slackSend (color: 'warning', message: "Delpoy aguardando aprovação: ${JOB_URL}", tokenCredentialId: 'Slack')
        			timeout(time: 10, unit: 'MINUTES') {
        				input(id: "Deploy Gate", message: "Deploy em homologação?", ok: 'Deploy')
        			}
        		}
        	}
        }
        stage ('Run Next Stage') {
                steps {
                    script {
                        try {
                            build job: 'vuttr-hml', parameters: [[$class: 'StringParameterValue', name: 'docker_image', value: 'jlosantana/vuttr-hml'],[$class: 'StringParameterValue', name: 'docker_container', value: 'vuttr-hml']]
                        } catch (e) {}
                    }
                }
            }
    }
}
```

### Executando tarefas paralelas

Casos de uso que não necessitam de execução sequencial das tarefas de pipeline podem ser beneficiados do uso de atividades paralelas, por exemplo, a publicação de um artefato gerado no processo de build em um repositório de artefatos (ex.: Artifactory), o deploy e a publicação da imagem no docker hub.

```
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/jlosantana/vuttr.git'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn -DskipTests clean package"
            }
        }
        
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${docker_image} ."
            }
        }
        
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f ${docker_container}'
                    } catch (e) {}
                }
            }
        }
        
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name ${docker_container} -d -p 3000:3000 ${docker_image}'
            }
        }
        
        stage('Notify Team') {
            steps {
                slackSend(color: 'success', message: 'Aplicação disponível para testes em: http://localhost:3000', tokenCredentialId: 'Slack')
            }
        }
        
        stage ('User Approval') {
        	steps {
        		script {
        			slackSend (color: 'warning', message: "Deploy aguardando aprovação: ${JOB_URL}", tokenCredentialId: 'Slack')
        			timeout(time: 10, unit: 'MINUTES') {
        				input(id: "Deploy Gate", message: "Deploy em homologação?", ok: 'Deploy')
        			}
        		}
        	}
        }
        
        stage('Deploy') {
            parallel {
            	stage ('Run Next Stage') {
                    steps {
                        script {
                            try {
                                build job: 'vuttr-hml', parameters: [[$class: 'StringParameterValue', name: 'docker_image', value: 'jlosantana/vuttr-hml'],[$class: 'StringParameterValue', name: 'docker_container', value: 'vuttr-hml']]
                            } catch (e) {}
                        }
                    }
                }
             	stage('Publish Artifact') {
                	steps {
                    	echo 'sending artifact to repository...'
                	}
            	}
            	stage('Build and Publish Image') {
                    when {
                    	branch 'master'  //only run these steps on the master branch
                    }
                    steps {                        
                    	sh """
                    		docker build -t ${IMAGE} .
                    		docker tag ${IMAGE} ${IMAGE}:${VERSION}
                    		docker push ${IMAGE}:${VERSION}
                    	"""
                   }
                }
            }
        }
    }
}
```

### Definindo e utilizando variáveis

```
pipeline {
    agent any
    
    environment {
    	// available in entire pipeline
    	image = "${docker_image}"
    	container = "${docker_container}"
  	}

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/jlosantana/vuttr.git'
            }
        }
        
        stage('Build') {
            steps {
                sh "mvn -DskipTests clean package"
            }
        }
        
        stage('Tests') {
            steps {
                sh "mvn test"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t $image ."
            }
        }
        
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f $container'
                    } catch (e) {}
                }
            }
        }
        
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name $container -d -p 3000:3000 $image'
            }
        }
        
        stage('Notify Team') {
        	environment {
                // only be available in this stage
                team_message = "Aplicação disponível para testes em: http://localhost:3000"
              }
            steps {
                slackSend(color: 'success', message: "$team_message", tokenCredentialId: 'Slack')
            }
        }
        
        stage ('User Approval') {
        	steps {
        		script {
        			slackSend (color: 'warning', message: "Deploy aguardando aprovação: ${JOB_URL}", tokenCredentialId: 'Slack')
        			timeout(time: 10, unit: 'MINUTES') {
        				input(id: "Deploy Gate", message: "Deploy em homologação?", ok: 'Deploy')
        			}
        		}
        	}
        }
        
        stage('Deploy') {
            parallel {
            	stage ('Run Next Stage') {
                    steps {
                        script {
                            try {
                                build job: 'vuttr-hml', parameters: [[$class: 'StringParameterValue', name: 'docker_image', value: 'jlosantana/vuttr-hml'],[$class: 'StringParameterValue', name: 'docker_container', value: 'vuttr-hml']]
                            } catch (e) {}
                        }
                    }
                }
             	stage('Publish Artifact') {
                	steps {
                    	echo 'sending artifact to repository...'
                	}
            	}
            	stage('Build and Publish Image') {
                    when {
                    	branch 'master'  //only run these steps on the master branch
                    }
                    steps {                        
                    	sh """
                    		docker build -t ${IMAGE} .
                    		docker tag ${IMAGE} ${IMAGE}:${VERSION}
                    		docker push ${IMAGE}:${VERSION}
                    	"""
                   }
                }
            }
        }
    }
    
    post {
        failure {
          // notify team when the Pipeline fails
          slackSend(color: 'error', message: "Pipeline fails! Build: ${currentBuild.fullDisplayName}. See ${env.BUILD_URL}", tokenCredentialId: 'Slack')
        }
  	}
}
```


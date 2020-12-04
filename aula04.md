[TOC]

------

# Aula 04

## Pipeline

### Utilizando Parâmetros 

Primeiramente definimos o build como parametrizado, então definimos duas variáveis:

* docker_image: Nome padrão da imagem docker. Valor padrão jlosantana/vuttr.

* docker_container: Nome padrão de execução do container. Valor padrão vuttr.

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
    }
}
```

### Integração com Slack

Vamos implementar uma integração de notificação com a ferramenta https://slack.com, para isso precisamos instalar o plugin https://plugins.jenkins.io/slack. Importante criar o seu Workspace, canal e app na ferramenta Slack seguindo a documentação https://slack.com/intl/pt-br/resources/slack-101 e https://mentoriafalar.slack.com/apps/A0F7VRFKN-jenkins-ci,  assim o Jenkins poderá enviar notificações diretamente para seu canal.

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
        // Notifica o time através do Slack
        stage('Notify Team') {
            steps {
                slackSend(message: 'vuttr disponível em: http://localhost:3000', tokenCredentialId: 'Slack')
            }
        }
    }
}
```

### Executando o Pipeline em um Docker Agent 

```
pipeline {
    agent {
        docker { image 'maven:3.6.3-jdk-11' }
    }
    stages {
        stage('Test Agent') {
            steps {
                sh 'mvn -v'
            }
        }
    }
}
```

#### Customizando nosso Docker Agent

Para compatibilidade com nosso Pipeline Script, a imagem docker precisa ser customizada para executar os passos de Checkout, Build e Tests. Para isso vamos estender a imagem padrão do Maven e adicionar o Git. 

##### Definindo o Dockerfile da nova imagem

```
FROM maven:3.6.3-jdk-11
RUN apt-get update && apt-get -y install git
```

##### Executando o build da nova imagem

```
docker build -t maven-git-agent .
```

##### Testando a nova imagem

```
pipeline {
    agent {
        docker { image 'maven-git-agent' }
    }
    stages {
        stage('Test Agent') {
            steps {
                sh 'mvn -v'
                sh 'git --version'
            }
        }
    }
}
```

#### Executando o Checkout, Build e Tests em um Docker Agent

```
pipeline {
    agent {
        docker { image 'maven-git-agent' }
    }
    stages {
        stage('Checkout') {
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
    }
}
```

### Utilizando o Jenkinsfile

Imagine que queremos customizar nosso Pipeline Script para diferentes ambientes por exemplo: dev, hml e prd. Para não precisarmos criar um job para cada branch, podemos utilizar a estratégia de criar um Multibranch Job onde cada branch terá seu próprio script pipeline.

#### Criando o Jenkinsfile de Dev

```
pipeline {
    agent any
    stages {        
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
                sh "docker build -t jlosantana/vuttr-dev ."
            }
        }        
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f vuttr-dev'
                    } catch (e) {}
                }
            }
        }        
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name vuttr-dev -d -p 3000:3000 jlosantana/vuttr-dev'
            }
        }        
    }
}
```


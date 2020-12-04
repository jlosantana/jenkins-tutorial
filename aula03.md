[TOC]

------

# Aula 03

## Pipeline

Jenkins Pipeline define um grupo de eventos encadeados que serão executados em um job/projeto. A diferença entre Jobs construídos com código e Jobs freestyle é a capacidade de ter maior flexibilidade e funcionalidades adicionadas pela linguagem de script.

Vamos utilizar uma aplicação Java disponível no repositório abaixo:

https://github.com/jlosantana/vuttr

Importante observar a documentação da aplicação para utilizarmos o Pipeline Script para executar todos os passos necessários para baixar o código, compilar, testar e executar.

### Pipeline simples

```
pipeline {
    agent any
    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

### Usando o Git

```
pipeline {
    agent any
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/jlosantana/vuttr.git'
            }
        }
    }
}
```

### Fazendo o Build

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
    }
}
```

### Executando Testes

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
    }
}
```

### Utilizando o Docker

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
                sh 'docker build -t jlosantana/vuttr .'
            }
        }
        // Para qualquer execução do Container em andamento
        stage('Stop Docker Container') {
            steps {
                script {
                    try {
                        sh 'docker rm -f vuttr'
                    } catch (e) {}
                }
            }
        }
        // Executa a Container da aplicação
        stage('Run Docker Container') {
            steps {
                sh 'docker run --name vuttr -d -p 3000:3000 jlosantana/vuttr'
            }
        }
    }
}
```


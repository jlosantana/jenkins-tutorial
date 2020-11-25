[TOC]

------

# Aula 02

## Gerenciamento

Acessando a interface gráfica do Jenkins em http://<host>:<port> podemos usar a conta do usuário Administrador para diversar funções de gerencianto e configurações.

### Conceitos Básicos

- **Job**: descrição de tarefas que o Jenkins deve executar.
- **Pipeline**: um modelo de execução definido que inclui o encadeamento de stages, steps e outros componentes.
- **Stages**: parte de um modelo de pipeline, define um conjunto de steps.
- **Steps**: uma tarefa específica que o Jenkins deve executar dentro de um modelo de pipeline.
- **Node**: uma máquina onde o jenkins pode executar suas tarefas.
- **Executor**: processo responsável pela execução de um job em um Node, podem existir mais de um executor para execução de jobs paralelos.
- **Workspace**: local no sistema de arquivos onde o Jenkins mantém arquivos utilizados durante a execução de um job.
-  **Build**: são as execuções dos jobs.
- **Credencials**: controla credenciais para acesso a sistemas de terceiros e integrações.
- **Plugin**: extensões utilizadas pelo Jenkins para adicionar novas funcionalidades.
- **REST API**: interface de aplicação que disonibiliza os objetos e funcionalidades para serem acessadas remotamente via protocolo HTTP.
- **Views**: disponibiliza uma organização do dashboard em abas.
- **CLI**: uma interface de linha de comando utilizada para executar operações e configurações.
- **Jenkinsfile**: arquivo que descreve um pipeline e que é colocado dentro do repositório juntamente com o código fonte do projeto.


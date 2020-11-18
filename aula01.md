[TOC]

------

# Aula 01

## Arquitetura

O Jenkins é uma ferramenta construída em Java para integração contínua. Ele é distribuído em formato executável compatível em qualquer plataforma que tenha o Runtime Java instalado.

## Instalação

### WAR file

```
wget https://get.jenkins.io/war-stable/2.249.3/jenkins.war
```

```
java -jar jenkins.war
```

Importante lembrar que por se tratar um pacote java para a web (WAR - Java Web Archive), o Jnekins pode ser executado em qualquer Servlet Container (Tomcat, Wildfly, JBoss, Weblogic, GlassFish etc.).

Por padrão a consolse de acesso ficará disponível em http://localhost:8080.

### Configurações Básicas 

![img](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-01-unlock-jenkins-page.jpg)

A instalação inicial do Jenkins requer um desbloqueio manual do administrador, a senha de desbloquio solicitada é informada no log, quando utilizado a instalação pelo pacote WAR.

![img](https://www.jenkins.io/doc/book/resources/tutorials/setup-jenkins-02-copying-initial-admin-password.png)

Por padrão o usuário 'admin' utiliza a mesma senha de desbloqueio para acesso ao ambiente de administração web.

### Instalação de Plugins Básicos

![img](https://automatingguy.com/public/img/posts/jcasc-first-encounter/2-jenkins-setup-wizard-plugins-choice.jpg)

### Criação do Usuário Administrador

![img](https://assets.digitalocean.com/articles/jenkins-install-ubuntu-1804/jenkins_create_user.png)

## Execução

Como falado o Jenkins é Java Web Archive (WAR) e sendo executado sobre a pltaforma Java significa que podemos utilizar de customizações de execução via linha de comando definidas pelo próprio Jenkins como as mais conhecidas definidas pela JVM.

### Parâmetros de rede

| Command Line Parameter             | Description                                                  |
| ---------------------------------- | ------------------------------------------------------------ |
| `--httpPort=$HTTP_PORT`            | Runs Jenkins listener on port $HTTP_PORT using standard *http* protocol. The default is port 8080. To disable (because you’re using *https*), use port `-1`. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--httpListenAddress=$HTTP_HOST`   | Binds Jenkins to the IP address represented by $HTTP_HOST. The default is 0.0.0.0 — i.e. listening on all available interfaces. For example, to only listen for requests from localhost, you could use: `--httpListenAddress=127.0.0.1` |
| `--httpsPort=$HTTPS_PORT`          | Uses HTTPS protocol on port $HTTPS_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--httpsListenAddress=$HTTPS_HOST` | Binds Jenkins to listen for HTTPS requests on the IP address represented by $HTTPS_HOST. |
| `--http2Port=$HTTP_PORT`           | Uses HTTP/2 protocol on port $HTTP_PORT. This option does not impact the root URL being generated within Jenkins logic (UI, inbound agent files, etc.). It is defined by the Jenkins URL specified in the global configuration. |
| `--http2ListenAddress=$HTTPS_HOST` | Binds Jenkins to listen for HTTP/2 requests on the IP address represented by $HTTPS_HOST. |
| `--prefix=$PREFIX`                 | Runs Jenkins to include the $PREFIX at the end of the URL. For example, set *--prefix=/jenkins* to make Jenkins accessible at *http://myServer:8080/jenkins* |
| `--ajp13Port=$AJP_PORT`            | Runs Jenkins listener on port $AJP_PORT using standard *AJP13* protocol. The default is port 8009. To disable (because you’re using *https*), use port `-1`. |
| `--ajp13ListenAddress=$AJP_ADDR`   | Binds Jenkins to the IP address represented by $AJP_HOST. The default is 0.0.0.0 — i.e. listening on all available interfaces. |
| `--sessionTimeout=$TIMEOUT`        | Sets the http session timeout value to $SESSION_TIMEOUT minutes. Default to what webapp specifies, and then to 60 minutes |

### Parâmetros/Propriedades de Sistema

Propriedades de sistema são utilizadas pela JVM para habilitar configurações especiais que ficam geralmente escondidas.

Utilizamos as propriedades informando através da linha de comando `-Dproperty=value`para o comando `java` que inicia o Jenkins. Importante, os parâmetros são para a JVM, então deve ficar logo após o java. Por exemplo:

```
java -Djava.util.logging.config.file=<pathTo>/logging.properties -jar jenkins.war
```